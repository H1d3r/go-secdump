# go-secdump

## Description
Package go-secdump is a tool built to remotely extract hashes from the SAM
registry hive as well as LSA secrets and cached hashes from the SECURITY hive
without any remote agent and without touching disk.

The tool is built on top of the library [go-smb](https://github.com/jfjallid/go-smb)
and use it to communicate with the Windows Remote Registry to retrieve registry
keys directly from memory.

It was built as a learning experience and as a proof of concept that it should
be possible to remotely retrieve the NT Hashes from the SAM hive and the LSA
secrets as well as domain cached credentials without having to first save the
registry hives to disk and then parse them locally.

The main problem to overcome was that the SAM and SECURITY hives are only
readable by NT AUTHORITY\SYSTEM. However, I noticed that the local group
administrators had the WriteDACL permission on the registry hives and could
thus be used to temporarily grant read access to itself to retrieve the
secrets and then restore the original permissions.

## Credits
Much of the code in this project is inspired/taken from Impacket's secdump
but converted to access the Windows registry remotely and to only access the
required registry keys.

Some of the other sources that have been useful to understanding the registry
structure and encryption methods are listed below: 

https://www.passcape.com/index.php?section=docsys&cmd=details&id=23

http://www.beginningtoseethelight.org/ntsecurity/index.htm

https://social.technet.microsoft.com/Forums/en-US/6e3c4486-f3a1-4d4e-9f5c-bdacdb245cfd/how-are-ntlm-hashes-stored-under-the-v-key-in-the-sam?forum=win10itprogeneral

## Usage
```
Usage: ./go-secdump [options]

options:
      --host <target>       Hostname or ip address of remote server
  -P, --port <port>         SMB Port (default 445)
  -d, --domain <domain>     Domain name to use for login
  -u, --user <username>     Username
  -p, --pass <pass>         Password
  -n, --no-pass             Disable password prompt and send no credentials
      --hash <NT Hash>      Hex encoded NT Hash for user password
      --local               Authenticate as a local user instead of domain user
      --dump                Saves the SAM and SECURITY hives to disk and
                            transfers them to the local machine.
      --relay               Start an SMB listener that will relay incoming
                            NTLM authentications to the remote server and
                            use that connection. NOTE that this forces SMB 2.1
                            without encryption.
      --relay-port <port>   Listening port for relay (default 445)
      --socks-host <target> Establish connection via a SOCKS5 proxy server
      --socks-port <port>   SOCKS5 proxy port (default 1080)
  -t, --timeout             Dial timeout in seconds (default 5)
      --noenc               Disable smb encryption
      --smb2                Force smb 2.1
      --debug               Enable debug logging
      --verbose             Enable verbose logging
  -v, --version             Show version
```

## Examples

Dump registry secrets

```
./go-secdump --host DESKTOP-AIG0C1D2 --user Administrator --pass adminPass123 --local
```
