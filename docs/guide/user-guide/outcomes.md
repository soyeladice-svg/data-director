# Understanding outcomes

The contract defines **five** possible outcome statuses in its
[Outcome vocabulary](../../contract.md#outcomes). A run may be suspended while
waiting for an external step, so not every run has a final outcome yet.

| Outcome     | Meaning                                                         |
| ----------- | --------------------------------------------------------------- |
| `succeeded` | The agent returned a result, and the harness's checks passed.   |
| `abstained` | The agent declined to answer rather than guess.                 |
| `referred`  | The matter needs a person's judgement.                          |
| `failed`    | The run did not complete, or a check failed.                    |
| `suspended` | The run is waiting for something outside the harness.          |

The app shows, with each outcome, what it means and what to do next. The contract specifies
which statuses carry a payload or a problem; in the current implementation `suspended` is not
yet produced (see the [contract](../../contract.md#outcomes)).
