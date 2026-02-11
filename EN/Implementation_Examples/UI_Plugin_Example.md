# Implementation Example: UI Plugin

This document provides a specific code example showing how a UI plugin implements its interface.

---

```javascript
// Example: A TUI (Text-based UI) plugin implementing the core's bi-directional communication interface
class Terminal_UI_Plugin {

  /**
   * Constructor called when the plugin is loaded
   * @param {object} coreApi - API object provided by the Core for communication
   */
  constructor(coreApi) {
    this.core = coreApi;
    this.initListeners();
    this.render(); // Render initial interface
  }

  // Listen for events from the Core
  initListeners() {
    this.core.on('onNewMessage', (message) => {
      this.displayMessageInChatWindow(message);
    });

    this.core.on('onToolCallRequest', (request) => {
      this.showConfirmationDialog(request.toolName, request.args, (approved) => {
        // Send the user's decision back to the Core
        this.core.provideConfirmation(request.confirmationId, approved);
      });
    });

    this.core.on('onReadyForInput', () => {
      this.enableUserInput();
    });
  }

  // Handle user input
  handleUserInput(text) {
    // Submit user input to the Core
    this.core.submitUserInput(text);
  }

  // ... Specific UI rendering code (displayMessage, showConfirmationDialog, etc.) omitted here
  render() { /* ... */ }
  displayMessageInChatWindow(message) { /* ... */ }
  showConfirmationDialog(tool, args, callback) { /* ... */ }
  enableUserInput() { /* ... */ }
}

// Plugin entry point, exporting the plugin class
module.exports = Terminal_UI_Plugin;
```
