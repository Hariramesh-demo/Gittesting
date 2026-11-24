# Call Transaction

Executes a transaction code in the current SAP GUI window.

**Package:** `UiPath.UIAutomation.Activities`
**Category:** UI Automation.SAP
**Assembly:** `UiPath.UIAutomationNext.Activities`
**Class Name:** `UiPath.UIAutomationNext.Activities.NSAPCallTransaction`
**Required Scope:** `UiPath.UIAutomationNext.Activities.NApplicationCard`

## Properties

### Input

| Name | Display Name | Kind | Type | Required | Default | Placeholder | Description |
|------|-------------|------|------|----------|---------|-------------|-------------|
| `Transaction` | Transaction Code | InArgument | `string` |  |  |  | Transaction code to be executed in the current SAP GUI window. |
| `Prefix` | Prefix | InArgument | `string` |  |  |  | Prefix to be prepended to the transaction (/n call transaction in the same window (session), /o call transaction in an additional session) |
| `InUiElement` | Input element | InArgument | [`UiElement`](common/UiElement.md) |  |  |  | The Input UI Element defines the screen element that the activity will be executed on. |

### Common

| Name | Display Name | Kind | Type | Default | Description |
|------|-------------|------|------|---------|-------------|
| `ContinueOnError` | Continue on error | InArgument | `bool` |  | Continue executing the activities in the automation if this activity fails. The default value is False. |
| `Timeout` | Timeout | InArgument | `double` |  | The amount of time (in seconds) to wait for the operation to be performed before generating an error. The default value is 30 seconds. |
| `DelayAfter` | Delay after | InArgument | `double` |  | Delay (in seconds) after this activity is completed, before next activity starts. The default amount of time is 0.3 seconds. |
| `DelayBefore` | Delay before | InArgument | `double` |  | Delay (in seconds) to wait before executing this activity. The default amount of time is 0.2 seconds. |

## XAML Example

Run `uip rpa --help` to find the command that retrieves the default XAML for an activity, then run it with the class name `UiPath.UIAutomationNext.Activities.NSAPCallTransaction`.

## Notes

- Before automating SAP, read [`sap-guide.md`](../references/sap-guide.md).
- This activity must be placed inside a `UiPath.UIAutomationNext.Activities.NApplicationCard` scope.
- The activity waits until the session reports the transaction it names, and fails early only on a status-bar error it recognises. `/n` alone therefore never succeeds: it lands on a start screen that reports `SESSION_MANAGER`, `SMEN` or `S000`, and the activity times out. To leave a transaction, see [sap-guide.md § Other Gotchas](../references/sap-guide.md#other-gotchas).
- Calling `SESSION_MANAGER` is no way back to the start screen: from inside a transaction SAP answers it with a "Cannot start transaction SESSION_MANAGER" dialog, and the activity succeeds where the start screen reports `SESSION_MANAGER` and times out where it reports `SMEN`.
