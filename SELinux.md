# Create custom policy

### General specifications
Policy Statements: https://selinuxproject.org/page/PolicyStatements

Object Permissions: https://selinuxproject.org/page/ObjectClassesPerms

### docker-samba policy
On my server I am runing various Docker containers, specifically a syncthing container. In addition I am also using my server for different Samba shares.

My goal is to mount the host directory /mnt/basis/software/backups/syncthing to the syncthing container, so that all files that are synchronized are accessible under the Samba share "backups". 

I used the following `podman-compose.yaml` as a basis:
```yml
version: "3.9"

services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:1.27.2
    container_name: syncthing
    hostname: syncthing-docker
    environment:
      - TZ=Europe/Berlin
    volumes:
      - /mnt/basis/software/docker/syncthing:/config
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: always
```
The problem here was, that due to the restrictions imposed by SELinux, the podman container was not able to access the host directory. Specifically the container process, is labeled as `container_t` and can not access the directory labeled as `samba_share_t`.

SELinux denial decisions are cached in Access Vector Cache (AVC). These can be inspected via `ausearch -m avc -ts recent`. There you can see different aspects of a denial decision. It could look something like that:

```
time->Mon Jan 22 15:36:11 2024
[...]
exe="/usr/bin/syncthing" 
type=AVC 
msg=audit(1705934171.372:3691): avc:  denied  { read } for  pid=41064  
[...]
scontext=system_u:system_r:container_t:s0:c210,c918 tcontext=system_u:object_r:samba_share_t:s0 
tclass=dir 
permissive=0
```

Where:

- `exe` shows which process was denied the operation
- `avc:  denied  { read }` specifies the type of the operation that was denied
- `scontext=system_u:system_r:container_t:s0:c210,c918` specified the source context of the denied operation. In this case, this context belongs to the podman container
- `tcontext=system_u:object_r:samba_share_t:s0` specified the target context of the directory on my host which the container should access
- `tclass=dir` refers to the target class object for which the operation was denied. In this case to target class object was a directory
- `permissive` tells us whether or not SELinux was in permissive mode when the operation was denied

Based on these information, a specific SELinux policy can be created which modifies SELinux so that the denied operation will be permitted. For that, we will be using the `audit2allow` tool, like:

```
ausearch -m avc -ts recent | audit2allow -ar
```

However, an appropriate policy can also be manually defined, as done in the file `docker-samba.te`:
```
module docker-samba 1.0;

require {
	type container_t;
	type samba_share_t;
	type mnt_t;
	class dir { read write setattr getattr create rename open add_name rmdir remove_name lock watch};
	class file { read write setattr getattr create rename open lock unlink append};
}

#============= container_t ==============
allow container_t samba_share_t:dir { read write setattr getattr create rename open add_name rmdir remove_name lock watch};
allow container_t samba_share_t:file { read write setattr getattr create rename open lock unlink append};
allow container_t mnt_t:dir { setattr };
```

Based on this policy file, on can install the new policy using the following commands. To check if your specification was indeed correct, you can again inspect the SELinux denial messages using `ausearch` as described before.

```
sudo checkmodule -M -m -o docker-samba.mod docker-samba.te
sudo semodule_package -m docker-samba.mod -o docker-samba.pp
sudo semodule -i docker-samba.pp
```

## References
For detailed information please refer to official documentation of SELinux as well as the tools used here.
