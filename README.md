https://github.com/KVM-VMI/kvm-vmi


# ubuntu-24.04.04 server
通常インストール

sudo apt update
sudo apt install -y build-essential git dpkg-dev dwarves \
  libncurses-dev bison flex libssl-dev libelf-dev zstd

sudo apt install -y bc rsync pahole

sudo apt install -y debhelper dh-exec libudev-dev


# ソース取得を有効化（24.04） カーネル 6.8を取得するため
sudo sed -i 's/^Types: deb$/Types: deb deb-src/' /etc/apt/sources.list.d/ubuntu.sources
sudo apt update

# カーネル6.8.0 取得
mkdir ~/linux_build
cd ~/linux_build/
apt source linux-source-6.8.0
#apt source linux-image-unsigned-6.8.0-107-generic



# vmiパッチを当てる
#cp -r ../build/src/6.8.0 ./
git clone 

cp -r ./6.8.0/* linux-6.8.0

# UAPIだけ残す シンボリックリンクで参照するようにしておく
rm ./linux-6.8.0/include/linux/kvmi.h
ln -s ./linux-6.8.0/include/uapi/linux/kvmi.h ./linux-6.8.0/include/linux/kvmi.h



cd linux-6.8.0/



cp /boot/config-$(uname -r) .config
#cp /boot/config-6.8.0-107-generic .config

./scripts/config --disable SYSTEM_TRUSTED_KEYS
./scripts/config --disable SYSTEM_REVOCATION_KEYS
./scripts/config --module KVM
./scripts/config --module KVM_INTEL
./scripts/config --module KVM_AMD
./scripts/config --enable KVM_INTROSPECTION
./scripts/config --disable TRANSPARENT_HUGEPAGE
./scripts/config --set-str CONFIG_LOCALVERSION "-kvmi"
 make olddefconfig





# build deb 作成
make clean
make -j$(nproc) bindeb-pkg 2>&1 | tee ./build.log

# インストール
cd ..
sudo dpkg -i ./linux-image-6.8.0-kvmi_*.deb linux-headers-6.8.0-kvmi_*.deb 

