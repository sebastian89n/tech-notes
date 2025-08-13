**Replay** allows you to re-run a Pipeline build with modified Pipeline script code **without changing the code in your SCM**.  
When you open a completed Pipeline build in Jenkins, you can click the **"Replay"** link to view and edit the Groovy script (Jenkinsfile) used in that run. After making changes, Jenkins executes the modified script immediately, using the same build parameters and workspace state as the original run.

Replay is useful for quick experimentation, debugging, or hotfix testing, as it avoids committing changes to your repository.  
However, it’s **not persisted** — changes made during replay are not saved back to SCM, and future builds will still use the original Jenkinsfile unless the changes are committed to the repository.

**Tip:** For security reasons, Replay is only available to users with the **"Run Scripts"** permission, since it can execute arbitrary Groovy code.

**Replay vs. Rerun vs. Rebuild**

|Feature|Replay|Rerun (Re-run Pipeline)|Rebuild (Re-run Freestyle)|
|---|---|---|---|
|**Purpose**|Run the job with _modified_ Pipeline script code|Run the exact same Pipeline code and parameters|Run the exact same Freestyle job|
|**Code source**|Edited in-browser for that run only|Original Jenkinsfile from SCM|Original job configuration|
|**Persistence**|Changes are not saved to SCM|No changes to SCM|No changes to job config|
|**Use case**|Quick tests/debug without committing|Repeat a run to verify reproducibility|Repeat a build with same parameters|
|**Job type**|Pipeline only|Pipeline only|Freestyle only|
|**Permissions**|Requires “Run Scripts”|Requires build permission|Requires build permission|
#### Note on "Restart from Stage"

The **Restart from Stage** feature lets you resume a failed or stopped Pipeline from a specific stage instead of starting from scratch. This is useful in long-running pipelines where only a later stage failed (e.g., deployment) but earlier stages (e.g., build, test) were successful.

- Requires the **Pipeline: Declarative** plugin and a Declarative Pipeline with `stages { ... }` defined.
- Jenkins will reuse artifacts and workspace from the earlier stages where possible.
- You cannot change the Pipeline code before restarting — if you need to modify the script, use **Replay** instead.