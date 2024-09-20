How to (re)create a diff for the latest kernel:

```
docker run --rm quay.io/centos/centos:stream9 bash -c '
set -Eeuo pipefail
dnf upgrade -y &>/dev/null
dnf install -y epel-release dnf-plugins-core xz diffutils &>/dev/null
crb enable >/dev/null
cd
rpm=$(dnf download --source kernel | grep -Eo "^kernel.*\\.rpm")
version=${rpm#kernel-}
version=${version%.src.rpm}

patched_dir=centos-9-stream-${version}
mkdir "$patched_dir"
rpm -i $rpm 2>/dev/null
tar xf rpmbuild/SOURCES/linux-${version}.tar.xz --strip-components=1 -C "$patched_dir"

major=${version%%.*}
minor=${version%.*.*}
minor=${minor#*.}
vanilla_url=https://cdn.kernel.org/pub/linux/kernel/v${major}.x/linux-${major}.${minor}.tar.xz
vanilla_dir=linux-${major}.${minor}-vanilla
mkdir "$vanilla_dir"
curl -fsSL "$vanilla_url" | tar Jx --strip-components=1 -C "$vanilla_dir"

diff -ur "$vanilla_dir" "$patched_dir"
' >kernel.patch
```
