## 2026-10-05

### Done
- Reviewed CoordinatorDistributed
- Reviewed DistributedTask
- Reviewed PingMessage
- Identified host/port based communication
- Identified distributed command flow
### Next
- Find how DistributedDaemon receives and handles commands
- Understand the behavior of each DS_* command
- Review AbstractSimulator
- Clarify the purpose of the mainPort and auxPort
- Clarify how the message flow is manage, pretty sure that the coordinator receives and sends, but I need to confirm if this is true and check if the simulators send messages between them
### Blockers / Questions
- Why are mainPort and auxPort both needed?
- Does simulator-to-simulator communication occur directly?
- Where is DistributedDaemon instantiated?