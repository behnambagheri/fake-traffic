#!/bin/bash

basename="$(basename "$(test -L "$0" && readlink "$0" || echo "$0")")"
mPWD=$PWD


log() {
    local string
    TIME_STAMP=$(date "+%F-%H-%M-%S")
    string="$*"
    echo -e "$TIME_STAMP - $string"
}

usage() {
    echo -e "usage write asap"
}

permissions() {

    # Detect environments where $PATH does not include the sbin directories
    if ! grep -q sbin <<< "$PATH"; then
        echo "$PATH does not include sbin. Try using "su -" instead of 'su'."
        exit
    fi

    if [[ "$EUID" -ne 0 ]]; then
        echo "This installer needs to be run with superuser privileges."
        echo -e "Run the script as root until the installation is complete, after that use it with normal privileges."
        exit
    fi
}

log "===========    START    ===========\n"


detect_os() {
    # Detect OS
    # $os_version variables aren't always in use, but are kept here for convenience
    if grep -qs "ubuntu" /etc/os-release; then
        os="ubuntu"
        os_version=$(grep 'VERSION_ID' /etc/os-release | cut -d '"' -f 2 | tr -d '.')
    elif [[ -e /etc/debian_version ]]; then
        os="debian"
        os_version=$(grep -oE '[0-9]+' /etc/debian_version | head -1)
    elif [[ -e /etc/almalinux-release || -e /etc/rocky-release || -e /etc/centos-release ]]; then
        os="centos"
        os_version=$(grep -shoE '[0-9]+' /etc/almalinux-release /etc/rocky-release /etc/centos-release | head -1)
    elif [[ -e /etc/fedora-release ]]; then
        os="fedora"
        os_version=$(grep -oE '[0-9]+' /etc/fedora-release | head -1)
    else
        echo "This installer seems to be running on an unsupported distribution.
    Supported distros are Ubuntu, Debian, AlmaLinux, Rocky Linux, CentOS and Fedora."
        exit
    fi
}

os_version() {
    if [[ "$os" == "ubuntu" && "$os_version" -lt 1804 ]]; then
        echo "Ubuntu 18.04 or higher is required to use this installer.
    This version of Ubuntu is too old and unsupported."
        exit
    fi

    if [[ "$os" == "debian" && "$os_version" -lt 9 ]]; then
        echo "Debian 9 or higher is required to use this installer.
    This version of Debian is too old and unsupported."
        exit
    fi

    if [[ "$os" == "centos" && "$os_version" -lt 7 ]]; then
        echo "CentOS 7 or higher is required to use this installer.
    This version of CentOS is too old and unsupported."
        exit
    fi
}

log_directory(){
    if ! [ -d /var/log/fake-traffic ]; then
        mkdir /var/log/fake-traffic &> /dev/null
        chmod -R 777 /var/log/fake-traffic
        echo -e "Log Directory created."

    fi
}

install_dependencies() {
    local dependencies
    dependencies=(vnstat jq tar wget)
    for i in "${dependencies[@]}";
    do

        if ! [[ $(command -v "$i") ]] ; then
            echo -e "Install PKG: $i"
            permissions

        #    echo -e "go not installed"
            if [[ "$os" = "debian" ]]; then

                apt-get update
                apt-get install -y  golang-go gcc git vnstat jq

            elif [[ "$os" = "ubuntu" ]]; then
                apt-get update
                apt install -y  golang-go gcc git vnstat jq

            elif [[ "$os" = "centos" ]]; then
                yum install -y epel-release
                yum install -y go gcc git vnstat jq
            else
                # Else, OS must be Fedora
                dnf install -y go gcc git vnstat jq
            fi
        fi
    done

    if command -v vnstat &> /dev/null; then
        systemctl enable --now vnstat
    fi

    if ! command -v librespeed-cli &> /dev/null; then
        permissions
        cd /tmp/ || exit

        install(){
            if tar tf librespeed-cli.tar.gz &> /dev/null; then
                tar xzvf librespeed-cli.tar.gz &> /dev/null
                cp librespeed-cli /usr/local/bin/librespeed-cli
                ln -s /usr/local/bin/librespeed-cli /bin/librespeed-cli
                echo -e "speedtest-cli installed successfully."
            else
                echo -e "librespeed-cli install unsuccessful, you should install it manually."
                echo -e "Follow the instruction on https://github.com/librespeed/speedtest-cli "
                exit 1
            fi
            }





        local architecture github_url release_on_ir_server url_prefix ir_url github_release ir_server_file uname
        uname=$(uname -s)
        architecture=""
        case $(lscpu | awk '/Architecture:/{print $2}') in
            i386)   architecture="386" ;;
            i686)   architecture="386" ;;
            x86_64) architecture="amd64" ;;
            arm)    architecture="arm64" ;;
        esac

        echo -e "System architecture is: $architecture"

        github_release=$(curl -s https://api.github.com/repos/librespeed/speedtest-cli/releases/latest \
            | grep "browser_download_url" \
            | cut -d : -f 2,3 \
            | tr -d \" \
            | grep $architecture \
            | grep -i "$uname" \
            | wget -O librespeed-cli.tar.gz -qi -)

        if ! [ -e librespeed-cli.tar.gz ]; then
            release_on_ir_server=$(curl -s https://drive.bea.sh/git-releases/librespeed-cli/ | cut -d'"' -f2 | grep -i "$(uname -s)" | grep $architecture)
    #        echo "release_on_ir_server: $release_on_ir_server"
            url_prefix='https://drive.bea.sh/git-releases/librespeed-cli/'
            ir_url="$url_prefix""$release_on_ir_server"
    #        github_release=$(curl -s "$github_url" -o librespeed-cli-github.tar.gz)
            ir_server_file=$(curl -s "$ir_url" -o librespeed-cli.tar.gz)
        fi


        if [ -e librespeed-cli.tar.gz ]; then
            install
        else
            echo -e "librespeed-cli install unsuccessful, you should install it manually."
            echo -e "Follow the instruction on https://github.com/librespeed/speedtest-cli "
            exit 1
        fi
    fi

}

install_script() {

    if ! command -v "$basename" &> /dev/null; then
        permissions

        cp "$mPWD"/"$0" /usr/local/bin/
        ln -s /usr/local/bin/"$basename" /bin/
        chmod +x /usr/local/bin/"$basename"

        if [ -e /bin/"$basename" ]; then
            log "\n\nThe Script installed successfully, you can run again without sudo"
            log_directory
        fi
    else
        if diff "$0" /bin/"$basename" &> /dev/null; then
            if [ -e /bin/"$basename" ]; then
#                log "\n\nThe Script already installed., you can run again without sudo"
                echo "silence is gild" &> /dev/null
                log_directory
            fi

        else
            cp "$0" /usr/local/bin/
            ln -s /usr/local/bin/"$basename" /bin/ &> /dev/null
            chmod +x /usr/local/bin/"$basename"

            if [ -e /bin/"$basename" ]; then
                log "\n\nThe Script updated successfully., you can run again without sudo"
                log_directory
            fi
        fi
    fi

}

update(){
    permissions
    curl -L short.platonic.ir/ft -o /tmp/fake-traffic

    if diff "$0" /tmp/"$basename" &> /dev/null; then
        if [ -e /bin/"$basename" ]; then
            log "\n\nThe latest Script already installed., you can run again without sudo"
            log_directory
            exit 0
        fi
    else
        cp /tmp/"$basename" /usr/local/bin/
        chmod +x /usr/local/bin/"$basename"

        if [ -e /bin/"$basename" ]; then
            log "\n\nThe Script updated successfully., you can run again without sudo"
            log_directory
            exit 0
        fi
    fi
}



run() {
       local random_method tts srt
       srt=$(shuf -i 15-120 -n1)
       random_method=$(shuf -i 1-2000 -n 1)
       tts=$(shuf -i 8-60 -n 1)

    if [ "$1" != "--no-sleep" ]; then
        log "The script will run after $srt minutes.\n"
        sleep "$srt"m
    fi


    if [ $((random_method%2)) -eq 0 ]; then
#        echo -e "Download mode"
        librespeed-cli --no-upload --simple --no-icmp --duration "$tts"
    else
#        echo -e "Upload mode"
        librespeed-cli --no-download --simple --no-icmp --duration "$tts"
    fi
}

stats () {
    permissions
    local rx_in_bytes rx_in_megabytes rx_in_gigabytes \
    tx_in_bytes tx_in_megabytes tx_in_gigabytes \
    tx10_rx1
    systemctl reload vnstat
    for i in $(vnstat --json | jq '.interfaces[].traffic.total.rx'); do

        if [ -z "$rx_in_bytes" ]; then
            rx_in_bytes=0
        fi
        rx_in_bytes=$((i + rx_in_bytes))
    done
    echo -e "RX in Bytes is: $rx_in_bytes"' B'
    rx_in_megabytes=$((rx_in_bytes / 1024 / 1024))
    echo -e "RX in MegaBytes is: $rx_in_megabytes"' MB'

    rx_in_gigabytes=$((rx_in_bytes / 1024 / 1024 / 1024))
    echo -e "RX in GigaBytes is: $rx_in_gigabytes"' GB'

    for i in $(vnstat --json | jq '.interfaces[].traffic.total.tx'); do

        if [ -z "$tx_in_bytes" ]; then
            tx_in_bytes=0
        fi
        tx_in_bytes=$((i + tx_in_bytes))
    done
    echo -e "TX in Bytes is: $tx_in_bytes"' B'
    tx_in_megabytes=$((tx_in_bytes / 1024 / 1024))
    echo -e "TX in MegaBytes is: $tx_in_megabytes"' MB'

    tx_in_gigabytes=$((tx_in_bytes / 1024 / 1024 / 1024))
    echo -e "TX in GigaBytes is: $tx_in_gigabytes"' GB'

    tx10_rx1=$((tx_in_bytes / rx_in_bytes))
    echo -e "tx10_rx1: $tx10_rx1"

    result=$tx10_rx1


}


run_base_stats() {

    stats

    local tts
    tts=$(shuf -i 44-280 -n 1)


    if [[ "$result" -gt 10 ]]; then
#        echo -e "Download mode"
        echo -e "Run speedtest in download mode with tts: $tts "
        librespeed-cli --no-upload --simple --no-icmp --duration "$tts"
    fi
    if [[ "$result" -lt 10 ]]; then
        #        echo -e "Upload mode"
        echo -e "Run speedtest in upload mode with tts: $tts "
        librespeed-cli --no-download --simple --no-icmp --duration "$tts"
    fi
    if [[ "$result" -eq 10 ]]; then
        echo -e "traffic is good."
    fi

}


case $1 in
    --install|install)
        detect_os
        os_version
        install_dependencies
        install_script
        ;;
    --update|update)
        update
        ;;
    --run|run)
        run
        ;;
    --10to1|10to1)
        detect_os
        os_version
        install_dependencies
        install_script
        run_base_stats #"$1"
        ;;
    --stats|stats)
        stats
        ;;
    --no-sleep|no-sleep)
        run "$1"
        ;;
    "")
        detect_os
        os_version
        install_dependencies
        install_script
        run
        ;;
    *)
        usage
        ;;

esac



echo -e "\n$TIME_STAMP - ===========     END     ==========="
echo -e "\n#########################################################\n"

