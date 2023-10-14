# syntax=docker/dockerfile:1.6

FROM navylinux/base

LABEL org.opencontainers.image.authors="Navy Linux Team <team@navylinux.org>"

WORKDIR /opt

ARG PYTHON_V 3.12
ARG PYTHON_SEMV 3.12.0

RUN <<EOF
#!/usr/bin/env bash
set -ex

dnf update -y
dnf -y groupinstall "Development Tools"

dnf install -y \
    bzip2-devel \
    curl \
    libffi-devel \
    openssl-devel \
    wget \
    xz-devel

# python full version (e.g., 3.12.0)
version=$PYTHON_SEMV

# python major.minor version (e.g., 3.12)
minor=$(echo ${version} | cut -d '.' -f 1,2)

# strip dot for href (312)
min=${minor/./}

# if minor version ends with a 0, add 0 to min (3120)
[[ ${version} =~ \.0$ ]] && min="${min}0"

# extract href
href=$(curl -s https://www.python.org/downloads/source/ | awk "/python-${min}/ {print \$0; exit}" | grep -oP '(?<=href=")[^"]*(?=")')

# combine with domain
url="https://www.python.org${href}"

# tgz exit on first match
tgz=$(curl -s ${url} | grep -oP '(?<=href=")[^"]*(?=")' | grep -E "Python-${version}.tgz" | head -n 1)

# download
cd /tmp
curl -s -O "$tgz"

# extract
tar -xzf Python-${version}.tgz

# build
cd Python-${version}
./configure --enable-optimizations
make -j $(nproc)

# install
sudo make altinstall

# unlink if exists
[[ -e "/usr/bin/python" ]] && sudo unlink /usr/bin/python
[[ -e "/usr/bin/pip" ]] && sudo unlink /usr/bin/pip

# symlink
sudo ln -s /usr/local/bin/python${minor} /usr/bin/python
sudo ln -s /usr/local/bin/pip${minor} /usr/bin/pip

# verify
python --version
pip --version

# remove build files and tgz
sudo rm -rf /tmp/Python-${version}*
EOF

CMD ["/usr/sbin/init"]
