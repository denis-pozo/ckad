Create a pod called `time-check` in the `dv11987` namespace. This pod should run
a container called `time-check` that uses the `busybox` image.

1. Create a config map called `time-config` with the data `TIME_FREQ=10` in the
same namespace.
2. The `time-check` container should run the command:
   ```shell
   while true; do date;
     sleep $TIME_FREQ
   done
   ```
   and write the result to the location `/opt/time/timecheck.log`.
3. The path `/opt/time` on the pod should mount a volume that lasts the lifetime
of this pod.
