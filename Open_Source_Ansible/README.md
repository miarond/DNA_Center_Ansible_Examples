# Open Source Ansible Examples

## Note to MacOS Users:

MacOS users may encounter an error similar to this, when running a DNA Center Ansible Playbook:

```
objc[61816]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called.
objc[61816]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead.  Set a breakpoint on objc_initializeAfterForkError to debug.
```
If this occurs, run the following command before attempting to run the Ansible Playbook again:

`export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES`