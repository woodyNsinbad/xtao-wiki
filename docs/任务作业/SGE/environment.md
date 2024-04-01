## Environment in XTAO SGE

### Public

- JOB_ID
    - id of job

- JOB_NAME
    - name of job

- HOME
  - home path of current user

- SGE_O_HOME
  - home path of current user

- SGE_O_HOST
    - host that submit job

- SGE_O_WORKDIR
    - workdir path of job

- SGE_CWD_PATH
    - current path. In general, same as workdir

- JB_EXEC_FILE (undefined)

- PE
  - name of PE

- NSLOTS
    - processes number(cpu) of specified PE

- SGE_STDOUT_PATH
  - stdout path of job

- SGE_STDERR_PATH
  - stderr path of job

- REQUEST
  - name of job or name of script file

- SGE_TASK_ID
    - task id of array job

- SGE_TASK_FIRST
  - the first(min) task id of array job

- SGE_TASK_LAST
  - the last(max) task id of array job

- SGE_TASK_STEPSIZE
  - the step size of array job