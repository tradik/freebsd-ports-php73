# Ports for php73 isolated from official repository just before removal.

It is risky to have a software that is no longer supported on a servers. Please notes/repository on your own risk and consider upgrading php software to a higher supported version.

## How to make it work:

1. Make sure poudriere-devel is installed as it has -O option
   ```
   pkg install poudriere-devel -y
   ```
2. Create a new jail for php73 packages
   ```
   /usr/local/bin/poudriere jail -c 130amd64-php73
   ```
3. Create a new repository in poudriere
   ```
   poudriere ports -c -p php73 -U git@github.com:tradik/freebsd-ports-php73.git -m git+ssh -v
   ```
4. Make sure you have options set up in /usr/local/etc/poudriere.d/options for php73 packages, manual step here - you can check example at `options` folder. You can copy this folder.
   ```
   ls -l /usr/local/etc/poudriere.d/options | grep php73
   cp -R /usr/local/poudriere/ports/php73/options /usr/local/etc/poudriere.d/130amd64-php73-options
   ```
5. Copy `130amd64-php73-make.conf` with default php73 setting in it
   ```
   cp /usr/local/poudriere/ports/php73/130amd64-php73-make.conf /usr/local/poudriere.d/
   ```
6. Update all ports trees
   ```
   /usr/local/bin/poudriere ports -u
   ```
7. Remove 'php73' MOVED references (tmp) from main REPO ( temporally for a build only )
   ```
   mv /usr/local/poudriere/ports/default/MOVED /usr/local/poudriere/ports/default/MOVED.old
   cat /usr/local/poudriere/ports/default/MOVED.old | grep -v php73 >/usr/local/poudriere/ports/default/MOVED
   ```
8. Build packages with list from `packages-php73`
   ```
   /usr/local/bin/poudriere bulk -f packages-php73 -O php73 -j 130amd64-php73
   ```
9. Bring back original MOVED. Otherwise you will get git stage error when you will be updating ports.
   ```
   mv /usr/local/poudriere/ports/default/MOVED.old /usr/local/poudriere/ports/default/MOVED
   ```

----

## How this repository was created:

1. Get last php73 working repo snapshot:
   ```
   mkdir php73-lastcommit
   git clone https://git.FreeBSD.org/ports.git
   git revert a60d5b0d747489751077ce93e8e52b44910f8eba
   ```
2. Create a this repository
   ```
   mkdir php73
   git init
   ```
   - Copy all php73 ports from the above repository to the new one and push.
      ```
      cat /builders/build-packages.list | grep php73 > packages-php73
      cat  packages-php73 | xargs cp php73-lastcommit/ports/{} php73/
      ```
   - Copy Mk php.mk definitions
      ```
      cp  php73-lastcommit/ports/Mk/Uses/php.mk .
      ```
   - Commit changes
      ```
      git add *
      git commit -m 'freebsd-ports-unsupported-php73'
      git push
      ```



