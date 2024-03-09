#!/bin/bash

# the behavior of this launcher can be adjusted by uncommenting and modifying the following values
# in the future, this will likely be replaced by an interface within Stencyl
# _MAX_MEM_="4096"
# _EXTRA_JVM_ARGS_=""
# _PRESERVE_TERMINAL_="false"

if [[ $(uname -m) == 'arm64' ]]; then
  jdkType="mac-aarch64"
else
  jdkType="mac64"
fi

cd "$(dirname "$0")"

qf="$(xattr -p com.apple.quarantine lib/sw.jar)"
if [ ! -z "$qf" ]; then
  xattr -dr com.apple.quarantine . > /dev/null 2>&1
fi

stencylData="$HOME/Library/Application Support/Stencyl"

if [ -z "$_MAX_MEM_" ]; then _MAX_MEM_="4096"; fi

checkJdk() { #path
    if [ -f "$1/bin/java" ] && "$1/bin/java" --version | grep "openjdk 21" > /dev/null; then
        echo "Found java at $1"
        export JAVA_HOME="$1"
        return 0
    fi
    return 1
}

downloadCheckJdk() { #remoteArchiveId, #localFolderID
    echo ""
    echo "Stencyl Java 21 runtime not found. It can be downloaded automatically."
    echo "[From URL: https://www.stencyl.com/dl/static/runtimes/stencyl/stencyl-jdk/$1]"
    echo "[To Folder: $stencylData/runtimes/stencyl-jdk/$2]"
    echo ""
    while true; do
        read -p "Would you like to download Java? (y/n) " yn
        case $yn in
            [Yy]* ) break;;
            [Nn]* ) return 1;;
            * ) echo "Please answer yes or no.";;
        esac
    done
    mkdir -p "$stencylData/temp/runtimes/stencyl-jdk/$1/.."
    mkdir -p "$stencylData/runtimes/stencyl-jdk/$2"
    rm -r "$stencylData/temp/runtimes/stencyl-jdk/$1"
    echo "Downloading Java runtime..."
    curl "https://www.stencyl.com/dl/static/runtimes/stencyl/stencyl-jdk/$1" --output "$stencylData/temp/runtimes/stencyl-jdk/$1" || return 1
    echo "Extracting Java runtime..."
    tar -xzf "$stencylData/temp/runtimes/stencyl-jdk/$1" -C "$stencylData/runtimes/stencyl-jdk/$2" --strip-components=1 || return 1
    rm "$stencylData/temp/runtimes/stencyl-jdk/$1"
    echo "Java is ready. Launching Stencyl."
    checkJdk "$stencylData/runtimes/stencyl-jdk/$2"
    return $?
}

launchStencyl() {

export SYSTEM_VERSION_COMPAT=0


if [ "$_PRESERVE_TERMINAL_" == "true" ]; then

"$JAVA_HOME/bin/java" \
 -Xmx${_MAX_MEM_}m \
 $_EXTRA_JVM_ARGS_ \
  --add-opens=java.desktop/com.apple.laf=ALL-UNNAMED \
 --add-opens=java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
 --add-opens=java.desktop/java.awt.dnd=ALL-UNNAMED \
 --add-opens=java.desktop/java.awt=ALL-UNNAMED \
 --add-opens=java.desktop/javax.swing=ALL-UNNAMED \
 --add-opens=java.desktop/sun.awt.datatransfer=ALL-UNNAMED \
 --add-opens=java.desktop/sun.awt.dnd=ALL-UNNAMED \
 --add-opens=java.desktop/sun.lwawt.macosx=ALL-UNNAMED \
 --enable-native-access=ALL-UNNAMED \
 --enable-preview \
 -Dapple.laf.useScreenMenuBar=true \
 -XX:-OmitStackTraceInFastThrow \
 -Xdock:icon="launcher/cog.icns" \
 -Xdock:name="Stencyl" \
 -Xms64m \
 -classpath "lib/*" \
 stencyl.sw.app.Launcher

else

nohup "$JAVA_HOME/bin/java" \
 -Xmx${_MAX_MEM_}m \
 $_EXTRA_JVM_ARGS_ \
  --add-opens=java.desktop/com.apple.laf=ALL-UNNAMED \
 --add-opens=java.desktop/com.sun.imageio.plugins.png=ALL-UNNAMED \
 --add-opens=java.desktop/java.awt.dnd=ALL-UNNAMED \
 --add-opens=java.desktop/java.awt=ALL-UNNAMED \
 --add-opens=java.desktop/javax.swing=ALL-UNNAMED \
 --add-opens=java.desktop/sun.awt.datatransfer=ALL-UNNAMED \
 --add-opens=java.desktop/sun.awt.dnd=ALL-UNNAMED \
 --add-opens=java.desktop/sun.lwawt.macosx=ALL-UNNAMED \
 --enable-native-access=ALL-UNNAMED \
 --enable-preview \
 -Dapple.laf.useScreenMenuBar=true \
 -XX:-OmitStackTraceInFastThrow \
 -Xdock:icon="launcher/cog.icns" \
 -Xdock:name="Stencyl" \
 -Xms64m \
 -classpath "lib/*" \
 stencyl.sw.app.Launcher >/dev/null 2>&1 &

sleep 1

fi

}

if checkJdk "runtimes/jre-$jdkType" && launchStencyl; then exit 0; fi
if checkJdk "$stencylData/runtimes/stencyl-jdk/21.0.1+12-$jdkType" && launchStencyl; then exit 0; fi
if downloadCheckJdk "21.0.1+12/stencyl-jdk-21.0.1+12-$jdkType.tar.gz" "21.0.1+12-$jdkType" && launchStencyl; then exit 0; fi

echo ""
echo "Failed to launch Stencyl."
echo ""
exit 1
