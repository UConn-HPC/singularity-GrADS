# -*- mode: rpm-spec -*-

# Released into the Public Domain:
# https://creativecommons.org/publicdomain/zero/1.0/legalcode

Bootstrap: docker
From: centos:7

%post
# Clear any previously installed files.
rm -rf /usr/local/*

# Install OS packages.
debug="strace"
dev="tar gzip"
deps="cairo libSM gd"
yum -y install $dev $deps $debug

# Install main executables and libraries.
url=ftp://cola.gmu.edu/grads/2.2/grads-2.2.0-bin-centos7.3-x86_64.tar.gz 
tarball=$(basename $url)
pkg=${tarball%-bin*}
test -f $tarball || curl -o $tarball $url
prefix=/usr
tar -xf $tarball
binaries=$(find $pkg/bin -type f -executable -exec basename {} \;)
cp -arv $pkg/bin $prefix
prefix=/usr/local/lib/grads
mkdir -p $prefix
cp -arv $pkg/lib/* $prefix

# Fail if any OS packages are missing, and suggest packages to install.
missing_libs=$(
    find $pkg -type f -executable -exec ldd {} + |
    grep 'not found' | awk '{print $1}')
if [ -n "${missing_libs}" ]; then
    echo >&2 "
Error: the following libraries are not yet installed:
$missing_libs

Suggesting packages to install...
"
    for lib in $missing_libs; do
        yum -q whatprovides *lib*/$lib
    done
    echo >&2 "
The above package set likely contains packages that depend on other packages.
To install the minimal required package set, check the output of each package
using repoquery.  Namely:

   repoquery --requires PACKAGE_NAME
"

    exit 1
fi

# Create the udpt file required by grads >= 2.1.
udpt=$prefix/udpt
rm -f $udpt
files=/usr/local/lib/grads/libgx*.so
for file in $files; do
    name=$(basename $file)
    name=${name%.so}
    name=${name#libgx}
    # libgxdummy is inconsistently named.
    test "$name" == "dummy" || name=${name:1}
    type="gxdisplay"
    if grep -q libgxp $file; then
        type="gxprint"
    fi
    echo "$type $name $file" >> $udpt
done
echo "Wrote udpt file:"
cat $udpt

# Install font and map data files.
url=ftp://cola.gmu.edu/grads/data2.tar.gz
tarball=$(basename $url)
test -f $tarball || curl -o $tarball $url
prefix=/usr/local/lib/grads
mkdir -p $prefix
tar -xvf $tarball -C $prefix

# Install examples.
url=ftp://cola.gmu.edu/grads/example.tar.gz
tarball=$(basename $url)
test -f $tarball || curl -o $tarball $url
prefix=/usr/local/share/grads/example
mkdir -p $prefix
tar -xvf $tarball -C $prefix

# Create startup message
cat <<EOF > /usr/local/share/grads/README.singularity
Welcome to the singularity image for GrADS!

Installed programs:
$binaries

To get help on any of the above programs, run

    singularity exec \$(which grads-2.2.0.img) PROGRAM -help

Or simply run the program of your choice

    singularity exec \$(which grads-2.2.0.img) PROGRAM

By default, "grads -l -b" will be run when you execute the image directly:

    grads-2.2.0.img
EOF

%test
exec grads -l -c QUIT

%runscript
cat /usr/local/share/grads/README.singularity
