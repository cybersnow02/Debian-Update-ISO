# Debian-Update-ISO
Make your own Debian Update ISO sets, so you can update your Debian Distribution faster without the need to download the whole Debian ISO sets again and again after each new release.

########################################################################
###   Debian - Build your own Update-ISO sets
###   Instructions - How-To-Do-It
########################################################################

# Why you need this?
# The Update-ISO sets is a way to update your actual Debian distribution
# with a newer one, WITHOUT downloading the whole new ISOs sets.
# These steps will create new ISOs that will superseed the ISO you
# already downloaded in the past. So it take alot less disk space
# because only the updated packages are included in the Update-ISOs set.
# So don't delete your older ISOs! You will still need them!
# As soon as you will have your new Update-ISOs built, you can
# mount/burn them and add them to your system manually with this command
# IN ROOT:
# mount debian-update-13.2.0-amd64-DVD-1.iso /media/cdrom
# apt-cdrom -d=/media/cdrom add
# mount debian-update-13.2.0-amd64-DVD-2.iso /media/cdrom
# apt-cdrom -d=/media/cdrom add
# mount debian-update-13.2.0-amd64-DVD-3.iso /media/cdrom
# apt-cdrom -d=/media/cdrom add
# ...
# And so on for all generated ISO...


# Before you start, you need to decide where your files will be
# downloaded on your computer. You must choose a partition/harddrive 
# that can handle gigabytes in size. You must have at least 60Gb
# or more. Newer Debian Release tend to grow in size so I suggest 100Gb.

# As soon as you decided where to put all the files, you can make the
# folder where you will put all the files. This is what we call
# 'the working place'. In this script, I've chosen :
# '/home/user/Desktop/debian' to be the 'the working place'.
# Replace that with your own path.

# The following steps refer to a Debian Trixie (13) distribution.
# If it is a newer or older release, you can replace any string 'Trixie'
# with the one you want to work with.

# The ISOs Updates will be built against AMD64 and I386 Architectures.
# If you want to use something else, see below.


# Let's start!


# We need to install the package 'debian-cd' and some download tools:
apt-get install debian-cd wget lftp


# All the scripts are installed in : /usr/share/debian-cd


# We need to create the directory structure ('the working place'):
# Change the paths according to what you have chosen earlier:
# Replace '/home/user/Desktop' with your own choice, somewhere that
# can handle 60Gb to 100Gb in size.
# Then, create a folder 'debian' and their sub-directories:
mkdir -p /home/user/Desktop/debian/ftp/debian/dists/trixie
mkdir -p /home/user/Desktop/debian/deb-cd/tmp/trixie-update/cd-work
mkdir -p /home/user/Desktop/debian/deb-cd/tmp/trixie-update/cd-out


# We need to download a 'DIFF' file, which permit to know the
# differences between the first release and the actual one.
# Visit this website: https://salsa.debian.org/images-team/package-lists
# For example, if the latest Debian version is 13.2, we need to download
# the file 'r0-r2.diff' under 'trixie' remote folder.
# Put this file under: /home/user/Desktop/debian/
cd /home/user/Desktop/debian/
wget https://salsa.debian.org/images-team/package-lists/-/raw/master/trixie/r0-r2.diff


# We need to modify some parameters/lines in the file:
# '/usr/share/debian-cd/update-cd'
# Open this file with your text editor.
# Change the following lines according to the wanted version and paths.
# If you are using cds, change 'dvd' for 'cd'
# The Architectures (Archs) used here are AMD64 and I386.
# You can decide to change that and use a supported one by Debian.

	MIRROR_NORM=/home/user/Desktop/debian/ftp/debian
	VER=13.2.0
	TDIR=/home/user/Desktop/debian/deb-cd/tmp/trixie-update/cd-work
	CODENAME=trixie
	OUT=/home/user/Desktop/debian/deb-cd/tmp/trixie-update/cd-out
	TYPE=dvd
	DIFF=/home/user/Desktop/debian/r0-r2.diff
	ARCHLIST="amd64 i386" # amd64 # -all dealt with specially


# By default, we remove the 'non-free' packages and unused Archs:
# The following commands keep only AMD64 and I386.
# If you have to change that, you must change the following command
# parameters and understand how they work (see manuals).
cd /home/user/Desktop/debian/
cat r0-r2.diff | grep -i -e "_amd64\.deb" -e "_i386\.deb" -e "_all\.deb" | grep -v "non-free\/" >to_download.txt
sed -i "s/^pool\//https:\/\/mirrors.glesys.net\/debian\/pool\//g" to_download.txt
cp r0-r2.diff r0-r2.diff.bak
rm r0-r2.diff
cat r0-r2.diff.bak | grep -i -e "_amd64\.deb" -e "_i386\.deb" -e "_all\.deb" | grep -v "non-free\/" >r0-r2.diff


# Now, we download all the packages needed to build the ISO sets:
# If you decide to stop/break the downloads, it will be resumed
# automatically with the same command.
# This will take alot of time depending of your Internet speed...
cd /home/user/Desktop/debian/ftp/debian
cat to_download.txt | shuf | xargs -n10 -P4 wget -x -nH --cut-dirs=1 --continue


# We need to download the remote structure/folder named 'indices':
cd /home/user/Desktop/debian/ftp/debian/
lftp -c 'set http:user-agent "Mozilla/5.0 (Windows; U; Windows NT 10.0; rv:97.0) Gecko/20100101 Firefox/97.0" ;set xfer:log 0 ;set xfer:log-file ;set ssl:verify-certificate false "/dev/null" ;mirror --skip-noaccess --parallel=100 --use-pget-n=10 https://mirrors.glesys.net/debian/indices/ ;exit'


# We need to download the remote structure/folder named 'dists' for the
# wanted architectures (AMD64 et iI386).
# Again, if you want to change the Archs, you must understand the
#  --exclude parameter (see the lftp manual)
cd /home/user/Desktop/debian/ftp/debian/dists/
lftp -c 'set http:user-agent "Mozilla/5.0 (Windows; U; Windows NT 10.0; rv:97.0) Gecko/20100101 Firefox/97.0" ;set xfer:log 0 ;set xfer:log-file ;set ssl:verify-certificate false "/dev/null" ;mirror --skip-noaccess --parallel=100 --use-pget-n=10 --exclude=binary-arm64 --exclude=binary-armel --exclude=binary-armhf --exclude=binary-ppc64el --exclude=binary-riscv64 --exclude=binary-s390x --exclude=by-hash --exclude=debian-installer --exclude=dep11 --exclude=i18n --exclude=source --exclude=installer- https://mirrors.glesys.net/debian/dists/trixie/ ;exit'


# We now start the script 'update-cd' to build our Updates ISO files:
# This will take some time...
/usr/share/debian-cd/update-cd


# When completed, the ISO files are in:
/home/user/Desktop/debian/deb-cd/tmp/trixie-update/cd-out/
