# 技術規格書 08: SecureStore 安全儲存指南

> **狀態**: Stable
> **適用版本**: OpenStarry v0.21.0-beta+
> **相關架構文檔**: 05_Security_and_Sandboxing_Protocol

本規格書旨在詳細定義 SecureStore 元件的架構、加密流程、併發控制機制，以及 API 使用規範，確保所有 Provider 插件能安全地儲存 API keys、OAuth tokens 等機敏資料。

---

## 1. 架構概覽

### 1.1 系統位置與目的

SecureStore 是 OpenStarry 框架中的加密儲存元件，位於核心套件 `@openstarry/shared` 的 `security/` 目錄：

```text
@openstarry/shared/
└── src/security/
    ├── secure-store.ts       # SecureStore 主類（含加密邏輯）
    └── file-lock.ts          # 雙層併發控制（in-process mutex + cross-process lock file）
```

**主要用途**：
- Provider plugins 的 API key 與 OAuth tokens 的加密儲存
- 跨機器不可解密的機敏資料保護（hostname + username 綁定）
- 並行寫入時的檔案一致性保證

### 1.2 獨立儲存隔離

每個 Plugin 擁有獨立的 `basePath`（由 PluginContext 注入），實現以下特性：

- **Plugin-level isolation**: 不同 Plugin 的儲存資料彼此獨立
- **Filesystem-level enforcement**: 基於 Unix chmod 或 Windows NTFS ACL
- **User-level enforcement**: 僅該用戶可讀寫其 Plugin 的儲存目錄

**典型路徑結構**：

```
$HOME/.openstarry/plugin-storage/
├── @openstarry-plugin/provider-gemini/
│   ├── credentials.json      # 加密儲存
│   ├── .lock                 # 檔案鎖（臨時）
│   └── state.json           # 狀態資料（可選）
└── @openstarry-plugin/provider-claude/
    └── credentials.json
```

---

## 2. 加密流程

### 2.1 加密演算法規範

SecureStore 使用 **AES-256-GCM** (Galois/Counter Mode)，提供認證加密（AEAD）：

| 參數 | 值 | 說明 |
|------|-----|------|
| **演算法** | AES-256-GCM | NIST 推薦的現代 AEAD 演算法 |
| **密鑰長度** | 256 bits (32 bytes) | AES-256 |
| **IV 長度** | 12 bytes (96 bits) | GCM 標準推薦（最快，避免認證標籤長度問題） |
| **認證標籤長度** | 16 bytes (128 bits) | 全長標籤，提供最強認證 |
| **Salt 長度** | 16 bytes (128 bits) | PBKDF2 最小推薦長度 |

### 2.2 密鑰推導（KDF）

密鑰來自環境資訊與 PBKDF2 推導：

**推導公式**：

```typescript
key = PBKDF2(
  password = hostname + username + saltSuffix,
  salt = 16 bytes random,
  iterations = 100,000,
  hashAlgorithm = SHA-512,
  keyLength = 32 bytes
)
```

**參數說明**：
- **password**: 由機器名稱、操作系統用戶名與 salt suffix 組成，以 `|` 分隔
  - 目的：不同機器/用戶解密不出資料（跨機器不可用）
  - 範例：`"linux-host.local|alice|openstarry"`
- **salt**: 每次寫入隨機生成，確保相同密碼產生不同密鑰
- **iterations = 100,000**: 抵禦字典攻擊（2024 OWASP 推薦值）
- **SHA-512**: 比 SHA-1 更強的雜湊演算法

### 2.3 EncryptedPayload 結構

加密資料的磁碟格式（JSON 編碼）：

```typescript
interface EncryptedPayload {
  // 初始化向量（hex 編碼）
  // 每次加密隨機生成 12 bytes
  iv: string;                // "a1b2c3d4e5f6g7h8i9j0k1l2"

  // GCM 認證標籤（hex 編碼）
  // 驗證密文完整性與 IV 一致性
  tag: string;               // "9f8e7d6c5b4a3b2c1d0e0f10"

  // 密鑰推導用的 salt（hex 編碼）
  // 與 IV 分離，因為 IV 在每次 read 時需要新生成
  salt: string;              // "f1a2b3c4d5e6f7a8b9c0d1e2"

  // Base64 編碼的密文
  // plaintext 經 AES-256-GCM 加密
  data: string;              // "ZXhhbXBsZUVuY3J5cHRlZERhdGE..."
}
```

**編碼層次**：
1. Plain object → JSON string
2. JSON string → UTF-8 bytes → AES-256-GCM encryption
3. (iv, tag, salt, ciphertext) → 分別 hex/base64 編碼 → JSON 物件
4. JSON 物件 → 寫入磁碟

**磁碟檔案內容範例**：

```json
{
  "iv": "a1b2c3d4e5f6g7h8i9j0k1l2",
  "tag": "9f8e7d6c5b4a3b2c1d0e0f10",
  "salt": "f1a2b3c4d5e6f7a8b9c0d1e2",
  "data": "ZXhhbXBsZUVuY3J5cHRlZERhdGE..."
}
```

### 2.4 檔案權限

**Unix (Linux/macOS)**：

```bash
chmod 600 $file   # rw------- (user only)
```

- 讀：owner 專有
- 寫：owner 專有
- 執行：禁止

**Windows (NTFS)**：

```powershell
icacls "$file" /inheritance:r /grant:r "$env:USERNAME`:F"
```

- 移除繼承的權限
- 當前用戶獲得完全控制（Full Control）
- 其他用戶無任何權限

---

## 3. File Lock 機制（v0.22.0-beta 新增）

### 3.1 雙層鎖設計

SecureStore 實作兩層併發控制，以支援多進程、多執行緒的並行操作：

#### Layer 1: In-Process Async Mutex

**目的**：序列化同一進程內的併發操作

**實現方式**：

```typescript
private locks = new Map<string, Promise<void>>();

private async acquireLock(filename: string): Promise<() => void> {
  // 等待前一個同名操作完成
  await this.locks.get(filename);

  let releaseLock: () => void;
  const lockPromise = new Promise<void>((resolve) => {
    releaseLock = resolve;
  });
  this.locks.set(filename, lockPromise);

  return () => releaseLock(); // 返回釋放函數
}

// 使用範例
async writeSecure<T>(filename: string, data: T): Promise<void> {
  const release = await this.acquireLock(filename);
  try {
    // 寫入操作
  } finally {
    release();
  }
}
```

**特點**：
- Per-file 粒度：不同檔名可並行；同一檔名序列化
- 非阻塞：使用 Promise 鏈，不佔用作業系統執行緒

#### Layer 2: Cross-Process Lock File

**目的**：防止多進程同時修改同一檔案

**實現方式**：

使用 OS-level 原子操作 `O_CREAT | O_EXCL`（檔案系統層級原子性）建立 `.lock` 檔：

```typescript
async function acquireFileLock(
  lockPath: string,
  timeout = 5000,
  staleMs = 30000,
  retryMs = 50
): Promise<() => Promise<void>> {
  const deadline = Date.now() + timeout;
  const maxBackoff = 200;
  let backoffMs = retryMs;

  while (Date.now() < deadline) {
    try {
      // 原子操作：檔案不存在才建立
      fs.openSync(lockPath, fs.constants.O_CREAT | fs.constants.O_EXCL);

      // 記錄 PID 與時戳
      const pidInfo = {
        pid: process.pid,
        timestamp: Date.now()
      };
      fs.writeFileSync(lockPath, JSON.stringify(pidInfo));

      // 成功取得鎖
      return async () => {
        // 釋放鎖：刪除檔案
        fs.unlinkSync(lockPath);
      };
    } catch (err) {
      if (err.code !== 'EEXIST') throw err;

      // Stale Lock Detection
      try {
        const pidInfo = JSON.parse(fs.readFileSync(lockPath, 'utf-8'));
        const age = Date.now() - pidInfo.timestamp;

        // 檢查 PID 是否存活 + 檔案過期
        if (age > staleMs && !isProcessAlive(pidInfo.pid)) {
          // 清除 stale lock
          fs.unlinkSync(lockPath);
          continue;
        }
      } catch {
        // 無法讀取 lock 檔，重試
      }

      // Exponential backoff
      await sleep(backoffMs);
      backoffMs = Math.min(backoffMs * 1.5, maxBackoff);
    }
  }

  throw new Error(`Failed to acquire lock after ${timeout}ms`);
}

function isProcessAlive(pid: number): boolean {
  try {
    // signal 0 = 檢查權限而不傳送信號
    process.kill(pid, 0);
    return true;
  } catch {
    return false;
  }
}
```

**特點**：
- 原子性：OS 保證 `O_CREAT | O_EXCL` 是原子的
- Stale detection：過期鎖（進程死亡 + 30s 未清理）自動清除
- Exponential backoff：避免 CPU 自旋（初始 50ms，上限 200ms）

### 3.2 Deadlock 預防

**設計規則**：

| 方法 | 加鎖 | 說明 |
|------|------|------|
| `write<T>()` | ✅ | 公開方法，需序列化 |
| `writeSecure<T>()` | ✅ | 涉及加密與 I/O |
| `delete()` | ✅ | 涉及檔案系統操作 |
| `readSecure<T>()` | ✅ | 涉及解密（legacy migration） |
| `read<T>()` | ❌ | 純讀取，無副作用 |
| `ensureDir()` | ❌ | 幂等操作，目錄層級（非檔案） |
| `_writeRaw()` (private) | ❌ | 內部方法，呼叫方負責加鎖 |
| `_deleteRaw()` (private) | ❌ | 內部方法，呼叫方負責加鎖 |

**無死鎖保證**：
1. 所有 public API 必須檢查是否需加鎖
2. Private/internal 方法不加鎖（減少嵌套鎖層級）
3. 鎖取得順序固定（filename alphabetical）

### 3.3 預設參數

```typescript
interface LockOptions {
  /**
   * 取得鎖的逾時時間（毫秒）
   * 預設值：5000 ms
   * 場景：等待其他進程釋放鎖的最長時間
   */
  timeout?: number;

  /**
   * Stale lock 偵測的年齡閾值（毫秒）
   * 預設值：30000 ms (30 秒)
   * 場景：如果鎖檔案超過 30 秒未更新，且 PID 已死亡，視為 stale
   */
  staleMs?: number;

  /**
   * 重試間隔初值（毫秒，exponential backoff）
   * 預設值：50 ms
   * 場景：首次重試等待 50ms，下次 75ms，...，上限 200ms
   */
  retryMs?: number;
}
```

---

## 4. API 參考

### 4.1 建構函數

```typescript
class SecureStore {
  constructor(options: SecureStoreOptions);
}

interface SecureStoreOptions {
  /**
   * 儲存檔案的基礎目錄
   * 通常由 PluginContext 提供：ctx.storagePath
   */
  basePath: string;

  /**
   * 密鑰推導附加字串（預設: "openstarry"）
   * 不同 saltSuffix 產生不同密鑰，實現 plugin 間的加密隔離
   */
  saltSuffix?: string;
}
```

### 4.2 公開方法

#### read<T>(filename: string): Promise<T | null>

**說明**：讀取未加密的 JSON 檔案

**簽名**：
```typescript
async read<T>(filename: string): Promise<T | null>
```

**行為**：
- 找不到檔案返回 `null`
- **不加鎖**（純讀取，無副作用）
- 拋出異常：檔案損壞或 JSON 解析失敗

**使用場景**：讀取非機敏的設定檔、狀態檔

**範例**：
```typescript
const config = await store.read<Config>('app-config.json');
if (!config) {
  console.log('No config found');
}
```

---

#### write<T>(filename: string, data: T): Promise<void>

**說明**：寫入未加密的 JSON 檔案（明文）

**簽名**：
```typescript
async write<T>(filename: string, data: T): Promise<void>
```

**行為**：
- 覆寫既有檔案或建立新檔案
- **加鎖**（序列化寫入）
- 寫入後設定檔案權限（chmod 600 / icacls）
- 目錄不存在時自動建立

**使用場景**：儲存非機敏的設定、快取、狀態

**範例**：
```typescript
await store.write('cache.json', { lastUpdate: Date.now() });
```

---

#### readSecure<T>(filename: string): Promise<T | null>

**說明**：讀取並解密的 JSON 檔案

**簽名**：
```typescript
async readSecure<T>(filename: string): Promise<T | null>
```

**行為**：
- 找不到檔案返回 `null`
- **加鎖**（因涉及解密與 legacy migration）
- 自動檢測舊版未加密資料，透明遷移為加密格式
- 解密失敗時：
  - 記錄警告訊息（`console.warn()`）
  - 刪除損壞的檔案（避免無限重試）
  - 返回 `null`

**使用場景**：讀取 API keys、OAuth tokens、密碼等機敏資料

**範例**：
```typescript
const creds = await store.readSecure<Credentials>('gemini-creds.json');
if (!creds) {
  console.log('No credentials found or decryption failed');
} else {
  console.log('API Key loaded:', creds.apiKey.substring(0, 5) + '...');
}
```

---

#### writeSecure<T>(filename: string, data: T): Promise<void>

**說明**：加密並寫入 JSON 檔案

**簽名**：
```typescript
async writeSecure<T>(filename: string, data: T): Promise<void>
```

**行為**：
- 序列化 `data` 為 JSON
- AES-256-GCM 加密 → EncryptedPayload 結構
- 寫入磁碟為 JSON 格式
- **加鎖**（序列化寫入）
- 設定檔案權限（chmod 600 / icacls）

**使用場景**：儲存 API keys、OAuth tokens、私密資料

**範例**：
```typescript
const credentials = {
  apiKey: 'sk-proj-abc123xyz...',
  refreshToken: 'token_xyz789...',
  expiresAt: Date.now() + 3600000
};

await store.writeSecure('gemini-creds.json', credentials);
```

---

#### delete(filename: string): Promise<void>

**說明**：刪除檔案

**簽名**：
```typescript
async delete(filename: string): Promise<void>
```

**行為**：
- 檔案不存在時**靜默忽略**（不拋出異常）
- **加鎖**（序列化刪除）

**使用場景**：撤銷授權、清除過期 token、卸載 plugin

**範例**：
```typescript
await store.delete('gemini-creds.json');
// 不拋出異常，即使檔案不存在
```

---

#### ensureDir(): Promise<void>

**說明**：確保 `basePath` 目錄存在

**簽名**：
```typescript
async ensureDir(): Promise<void>
```

**行為**：
- 若目錄不存在，建立它（包括父目錄）
- 若目錄已存在，無操作（幂等）
- **不加鎖**（目錄層級操作，非檔案級）
- **自動呼叫**：`write()` / `writeSecure()` 前會隱含呼叫

**使用場景**：初始化 plugin 儲存目錄

**範例**：
```typescript
await store.ensureDir();
// 現在 basePath 目錄肯定存在
```

---

### 4.3 方法摘要表

| 方法 | 說明 | 加鎖 | 機敏性 |
|------|------|------|--------|
| `read<T>(filename)` | 讀取明文 JSON | ❌ | 低 |
| `write<T>(filename, data)` | 寫入明文 JSON | ✅ | 低 |
| `readSecure<T>(filename)` | 讀取並解密 | ✅ | 高 |
| `writeSecure<T>(filename, data)` | 加密並寫入 | ✅ | 高 |
| `delete(filename)` | 刪除檔案 | ✅ | 任意 |
| `ensureDir()` | 確保目錄存在 | ❌ | 無 |

---

## 5. 安全考量

### 5.1 跨機器隔離

**設計目標**：在盜取磁碟後，其他機器無法解密資料

**實現方式**：
- 密鑰推導時綁定 hostname + username
- 即使攻擊者取得加密檔案，也必須知道原始機器的用戶名才能嘗試解密
- 不同機器 → 不同密鑰 → 無法解密

**範例**：
```
Machine A (hostname=server1, user=alice)
  → key = PBKDF2("server1:alice:$opensecure$", ...)
  → Encrypt with this key

Machine B (hostname=server2, user=bob)
  → key = PBKDF2("server2:bob:$opensecure$", ...)
  → 無法解密 Machine A 的資料（不同 password）
```

### 5.2 Stale Lock Handling

**問題**：程式崩潰後，`.lock` 檔可能永遠留下

**解決方案**：
1. Lock 檔記錄 PID 與時戳
2. 重試時檢查：
   - PID 是否存活（`process.kill(pid, 0)`）
   - 檔案是否超過 30 秒未更新
3. 如果 PID 已死亡 AND 超過 30 秒 → 清除舊鎖

**優勢**：無需外部 lock daemon，自動恢復

### 5.3 Legacy Migration

**場景**：升級前，使用者已儲存未加密的舊版 JSON 檔案

**自動遷移機制**：
```typescript
async readSecure<T>(filename: string): Promise<T | null> {
  const filePath = path.join(this.basePath, filename);

  try {
    const raw = fs.readFileSync(filePath, 'utf-8');
    const data = JSON.parse(raw);

    // 檢查是否是 EncryptedPayload 結構
    if (this.isEncryptedPayload(data)) {
      // 新版格式 → 直接解密
      return this.decrypt(data);
    } else {
      // 舊版明文 → 自動加密並覆寫
      console.warn(`Migrating ${filename} to encrypted format...`);
      await this.writeSecure(filename, data);
      return data as T;
    }
  } catch (err) {
    console.warn(`Failed to read/migrate ${filename}:`, err.message);
    // 檔案損壞，刪除它
    try { fs.unlinkSync(filePath); } catch {}
    return null;
  }
}
```

**好處**：
- 無縫升級體驗
- 舊資料自動加密
- 不需手動遷移腳本

### 5.4 解密失敗恢復

**現象**：GCM 認證標籤不匹配（資料被篡改或密鑰錯誤）

**處理**：
```typescript
try {
  const decrypted = decipher.update(ciphertext, 'hex', 'utf-8')
                          + decipher.final('utf-8');
  decipher.setAuthTag(tag);
} catch (err) {
  // 認證失敗 = 資料被篡改或密鑰錯誤
  console.warn(`Authentication failed for ${filename}. Deleting corrupted file.`);
  fs.unlinkSync(filePath);
  throw new SecurityError('Encrypted data authentication failed');
}
```

**不會靜默失敗**：必須明確通知應用層，應用層決定是否重新授權

---

## 6. 使用範例

### 6.1 Provider Plugin 的典型使用

**場景**：Gemini Provider 存儲 API key

```typescript
import { SecureStore } from '@openstarry/shared';

export async function createGeminiPlugin(ctx: IPluginContext): Promise<IPlugin> {
  const store = new SecureStore({
    basePath: ctx.storagePath,
    logger: ctx.logger
  });

  return {
    async initialize() {
      // 初始化時確保目錄存在
      await store.ensureDir();
    },

    async handleCliCommand(cmd: string, args: string[]): Promise<string> {
      if (cmd === 'provider gemini login') {
        const [apiKey] = args;

        // 檢查並儲存 API key
        if (!apiKey.startsWith('sk-')) {
          throw new Error('Invalid API key format');
        }

        // 加密儲存
        await store.writeSecure('credentials.json', { apiKey });
        return 'Gemini API key saved securely';
      }

      if (cmd === 'provider gemini logout') {
        // 刪除憑證
        await store.delete('credentials.json');
        return 'Gemini API key removed';
      }

      if (cmd === 'provider gemini status') {
        // 檢查是否已授權（不讀取實際 key）
        const creds = await store.readSecure('credentials.json');
        return creds ? 'Authenticated' : 'Not authenticated';
      }
    }
  };
}
```

### 6.2 Provider Chat 實現中的使用

**場景**：Chat 時動態讀取 API key

```typescript
export class GeminiProvider implements IProvider {
  private store: SecureStore;

  constructor(storagePath: string) {
    this.store = new SecureStore({ basePath: storagePath });
  }

  async chat(request: ChatRequest): Promise<AsyncIterable<ProviderStreamEvent>> {
    // 讀取加密的 API key
    const creds = await this.store.readSecure<{ apiKey: string }>(
      'credentials.json'
    );

    if (!creds?.apiKey) {
      throw new Error('Gemini API key not configured. Run: /provider gemini login <KEY>');
    }

    // 使用 API key 呼叫 Google AI API
    const response = await fetch('https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:streamGenerateContent', {
      method: 'POST',
      headers: {
        'X-Goog-Api-Key': creds.apiKey,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(this.buildRequestPayload(request))
    });

    return this.streamResponse(response);
  }
}
```

### 6.3 OAuth Token 更新流程

**場景**：Claude Provider 定期刷新 OAuth token

```typescript
export class ClaudeOAuthProvider implements IProvider {
  private store: SecureStore;

  constructor(storagePath: string) {
    this.store = new SecureStore({ basePath: storagePath });
  }

  private async refreshTokenIfNeeded(): Promise<string> {
    const creds = await this.store.readSecure<OAuthCredentials>(
      'oauth-creds.json'
    );

    if (!creds) {
      throw new Error('Not authenticated');
    }

    // 檢查是否即將過期（5 分鐘內）
    if (Date.now() + 300000 >= creds.expiresAt) {
      const newToken = await this.fetchNewAccessToken(creds.refreshToken);

      // 更新 token（加密儲存）
      await this.store.writeSecure('oauth-creds.json', {
        ...creds,
        accessToken: newToken.access_token,
        expiresAt: Date.now() + newToken.expires_in * 1000
      });

      return newToken.access_token;
    }

    return creds.accessToken;
  }

  async chat(request: ChatRequest): Promise<AsyncIterable<ProviderStreamEvent>> {
    const token = await this.refreshTokenIfNeeded();

    // 使用新 token 呼叫 Claude API
    // ...
  }
}
```

---

## 7. 限制與未來方向

### 7.1 當前限制

| 限制 | 原因 | 工作項 |
|------|------|--------|
| 不支援跨機器分散式鎖 | 需要外部 lock service | Plan07.3 + Redis |
| 無密鑰輪轉 | 複雜度高（需版本控制） | Plan08 |
| 無內建備份/恢復機制 | 超出本元件範圍 | 應用層實現 |
| 無自動 token 刷新 | 應用層責任 | Provider 實現 |

### 7.2 未來增強（v0.23.0-beta+）

**密鑰輪轉** (v0.23.0-beta 計畫)：
```typescript
interface EncryptedPayloadV2 extends EncryptedPayload {
  keyVersion: number;  // 支援多個密鑰版本
  rotatedAt?: number;  // 輪轉時戳
}

async rotateKeys(): Promise<void> {
  // 重新加密所有檔案
  // 舊密鑰逐步淘汰（grace period）
}
```

**Redis-backed 分散式鎖** (v0.23.0-beta):
```typescript
interface DistributedLockOptions {
  redisUrl?: string;  // Redis 連接 URL
  // 若設定，使用 Redis 實現分散式鎖
  // 若未設定，降級為檔案鎖（單機）
}
```

---

## 8. 測試清單

SecureStore 的測試應涵蓋：

- [ ] **基本 I/O**：write/read plain JSON
- [ ] **加密與解密**：writeSecure/readSecure 往返
- [ ] **Legacy 遷移**：舊版未加密資料自動升級
- [ ] **檔案權限**：寫入後權限正確（600 / icacls）
- [ ] **跨機器隔離**：不同 hostname/username 無法解密
- [ ] **併發寫入**：多執行緒同時 write 不會損壞資料
- [ ] **多進程競爭**：多進程同時寫入單一檔案
- [ ] **Stale lock 清理**：程式崩潰後自動釋放過期鎖
- [ ] **損壞檔案恢復**：解密失敗時靜默刪除並返回 null
- [ ] **目錄隔離**：不同 Plugin 的 basePath 隔離
- [ ] **磁碟滿錯誤**：處理寫入失敗

---

## 9. 相關文檔與引用

- **架構文檔**: `05_Security_and_Sandboxing_Protocol.md` — 安全模型與沙盒設計
- **原始碼**：`@openstarry/shared/src/security/secure-store.ts` — SecureStore 實作
- **鎖實作**：`@openstarry/shared/src/security/file-lock.ts` — 雙層鎖實作
- **Node.js Crypto API**：https://nodejs.org/api/crypto.html
- **PBKDF2 (RFC 8018)**：https://tools.ietf.org/html/rfc8018

---

## 10. 修改歷史

| 版本 | 日期 | 變更說明 |
|------|------|----------|
| 1.0 | 2026-02-15 | 初版發布，涵蓋 v0.21.0-beta 至 v0.22.0-beta |
| 1.1 (計畫) | 2026-03H | 加入密鑰輪轉規範、分散式鎖 |

---

**文檔終版** (Stable for v0.21.0-v0.22.0-beta)
