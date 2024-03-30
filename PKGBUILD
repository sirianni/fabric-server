# Maintainer: lgm <lgm dot aur at outlook dot com>
# Contributor: Ndoskrnl <lollipop.studio.cn@gmail.com>
# Contributor: flying <flyinghat42@gmail.com>

# Based on the 'forge-server' AUR package by:
# Maintainer: Nitroretro <nitroretro@protonmail.com>

# Based on the `minecraft-server` AUR package by:
## Maintainer: Gordian Edenhofer <gordian.edenhofer@gmail.com>
## Contributor: Philip Abernethy <chais.z3r0@gmail.com>
## Contributor: sowieso <sowieso@dukun.de>

_ver="1.20.4_1.0.0_0.15.7-0"
_minecraft_ver_latest="1.20.4"

IFS="-" read -ra _ver_temp <<< "$_ver"
IFS="_" read -ra _pkgver_temp <<< "${_ver_temp[0]}"

# the minecraft version
_minecraft_ver=${_pkgver_temp[0]}

# the version of the installer
_fabric_ver=${_pkgver_temp[1]}

# the version of the loader to install
_fabric_loader_ver=${_pkgver_temp[2]}

_mng_ver=1.0.4

_pkgver=${_ver_temp[0]//_/-}

if [ "$_minecraft_ver" = "$_minecraft_ver_latest" ]; then
	pkgname="fabric-server"
	_fabric_name="fabric"
else
	pkgname="fabric-server-${_minecraft_ver}"
	_fabric_name="fabric-${_minecraft_ver}"
fi

pkgver=${_ver_temp[0]}
pkgrel=${_ver_temp[1]}
pkgdesc="Minecraft Fabric server unit files, script and jar"
arch=("any")
url="https://fabricmc.net"
license=("Apache")
depends=("java-runtime-headless>=17" "tmux" "sudo" "bash" "awk" "sed")
optdepends=("tar: required in order to create world backups"
	"netcat: required in order to suspend an idle server")
provides=("fabric-server=${pkgver}")
backup=("etc/conf.d/${_fabric_name}")
install="fabric-server.install"

source=("minecraft-server-${_mng_ver}.tar.gz"::"https://github.com/Edenhofer/minecraft-server/archive/refs/tags/v${_mng_ver}.tar.gz"
		"fabric-installer-${_fabric_ver}.jar"::"https://maven.fabricmc.net/net/fabricmc/fabric-installer/${_fabric_ver}/fabric-installer-${_fabric_ver}.jar")
noextract=("fabric-${_pkgver}.jar")
sha512sums=('dd4d68ca061c97a1e3cb5c0bb68439f7d8d45b15092344f3c4dbd4f7f39fef433d566670ad440970061007d93055183b570c7bf98f09c111ecdf8ab0f208f556'
            '4436abb017554703ccb070620a9a39b33ccd380d8d640c88f8d3f5b434ec73bf95c3dbcc9824243dafe2f3bb19fc0f9b9a5f27c2896f8162838f297b77b581d4')

prepare() {
	java -Duser.home="${srcdir}" -jar "fabric-installer-${_fabric_ver}.jar" server -mcversion ${_minecraft_ver} -downloadMinecraft -loader ${_fabric_loader_ver}
}

_game="fabric"
_server_root="${pkgdir}/srv/${_fabric_name}"
build() {
	make -C "${srcdir}/minecraft-server-${_mng_ver}" \
		GAME=${_game} \
		INAME=${_game}d \
		SERVER_ROOT=${_server_root} \
		BACKUP_PATHS="world world_nether world_the_end" \
		GAME_USER=${_game} \
		MAIN_EXECUTABLE=fabric-server-launch.jar \
		SERVER_START_CMD="java -Dlog4j2.formatMsgNoLookups=true -Xms512M -Xmx1024M -jar './\$\${MAIN_EXECUTABLE}' nogui" \
		clean
	make -C "${srcdir}/minecraft-server-${_mng_ver}" \
		GAME=${_game} \
		INAME=${_game}d \
		SERVER_ROOT=${_server_root} \
		BACKUP_PATHS="world world_nether world_the_end" \
		GAME_USER=${_game} \
		MAIN_EXECUTABLE=fabric-server-launch.jar \
		SERVER_START_CMD="java -Dlog4j2.formatMsgNoLookups=true -Xms512M -Xmx1024M -jar './\$\${MAIN_EXECUTABLE}' nogui" \
		all
}

package() {
	_server_root="${pkgdir}/srv/${_fabric_name}"

	make -C "${srcdir}/minecraft-server-${_mng_ver}" \
		DESTDIR="${pkgdir}" \
		GAME=${_game} \
		INAME=${_game}d \
		install

	# Install Fabric
	install -Dm644 "fabric-server-launch.jar" "${_server_root}/fabric-server-launch.jar"

	# Install Minecraft Server
	install -Dm644 "server.jar" "${_server_root}/server.jar"

	# install the libraries subfolder
	# 1 create the emptyfolder structure 
	install -dm755 "libraries" "${_server_root}/libraries"
	for d in $(find "libraries" -type d);do
		install -d --mode 755 "$d" "${_server_root}/${d}";
	done
	# 2 install all files
	for f in $(find "libraries" -type f); do
		install -D --mode 755 "$f" "${_server_root}/${f}";
	done

	# Link log files
	mkdir -p "${pkgdir}/var/log/"
	install -dm2755 "${_server_root}/logs"
	ln -s "/srv/${_fabric_name}/logs" "${pkgdir}/var/log/${_fabric_name}"

	# Give the group write permissions and set user or group ID on execution
	chmod g+ws "${_server_root}"
}
