# FreeIPA generate keytab failed, missing cache file.

## Issue 

Failed when regenerate keytab file for services from Ambari Web UI.

First error logs in `Create Keytabs` task:

```
2023-01-04 09:03:58,057 - Failed to create keytab for hive/yava30w03l2tuner.syslabs247.com@SYSLABS247.COM, missing cached file
2023-01-04 09:03:58,057 - Failed to create keytab for hive/yava30w03l2tuner.syslabs247.com@SYSLABS247.COM, missing cached file
2023-01-04 09:03:58,064 - Failed to create keytab for oozie/yava30w02l2tuner.syslabs247.com@SYSLABS247.COM, missing cached file
2023-01-04 09:03:58,066 - Failed to create keytab for hive/yava30w03l2tuner.syslabs247.com@SYSLABS247.COM, missing cached file
2023-01-04 09:03:58,066 - Failed to create keytab for hive/yava30w03l2tuner.syslabs247.com@SYSLABS247.COM, missing cached file
```

After investigation, we found error log on Success `Create Principals` task :

```
2023-01-04 09:03:56,703 - Failed to create principal, hive/yava30w03l2tuner.syslabs247.com@SYSLABS247.COM - Failed to create principal for hive/yava30w03l2tuner.syslabs247.com@SYSLABS247.COM
STDOUT: 
STDERR: ipa: ERROR: Host 'yava30w03l2tuner.syslabs247.com' does not have corresponding DNS A/AAAA record

2023-01-04 09:03:56,703 - Failed to create principal,  - Failed to create new principal - no principal specified
2023-01-04 09:03:57,417 - Failed to create principal, oozie/yava30w02l2tuner.syslabs247.com@SYSLABS247.COM - Failed to create principal for oozie/yava30w02l2tuner.syslabs247.com@SYSLABS247.COM
STDOUT: 
STDERR: ipa: ERROR: Host 'yava30w02l2tuner.syslabs247.com' does not have corresponding DNS A/AAAA record
```


Environment :

- YAVA Data Platform 3.0
- FreeIPA 4.6.8


Resolution :

1. Make sure FreeIPA server using integrated DNS or external DNS which contain all YAVA cluster nodes A/AAAA record.
2. Setting correct DNS server for All Nodes including FreeIPA server.
3. Restart freeipa service using command `ipactl restart`.
4. Try again to regenerate keytab from Ambari Web UI.