# CubeSat Setup Procedures

## Installing .Net dependencies
**Note:** These instructions assume you are using the most recent Ubuntu LTS. Mileage may vary on other platforms.
They are more or less derived from the instructions found in [README.md](README.md), but may deviate in places. Ideally, the reasons for deviation will be documented here.

1. Install the .Net Framework SDK
    - `sudo apt update && sudo apt install -y dotnet-sdk-8.0`
    - The README mentions the use of Mono, which is no longer maintained and the work of which has been pulled under the umbrella of the Microsoft .Net Framework. Attempts to produce a working Mono build were unsucessful.
2. Install other depencies
    - **Note:** several of these packages may already be installed on your system. (For example: All Ubuntu WSL instances ship with policykit-1, python3, and screen)
    - `sudo apt install gcc gtk-sharp2 libc6-dev libgtk2.0-0 libicu-dev policykit-1 python3 python3-pip screen uml-utilities`

## Perform initial build
**Note:** These instructions assume you are using the most recent Ubuntu LTS. Mileage may vary on other platforms.
They are more or less derived from the instructions found in the [Renode Docs](https://renode.readthedocs.io/en/latest/advanced/building_from_sources.html), but may deviate in places. Ideally, the reasons for deviation will be documented here.

1. Install build dependencies
    - `sudo apt install autoconf automake cmake coreutils g++ libgtk2.0-dev libtool`
2. Run the build script in the root of the repo
    - **Note:** The `--net` flag is used to specify a dotnet build rather than a mono build.
    - `./build.sh --net`
    - Running this initial build with also initialize the git submodules
    - **Note:** If your build generates the warning:
        -`bash : warning : setlocale: LC_ALL: cannot change locale (en_US.UTF-8) [/home/hcorse/Code/Cubesat/renode/src/Renode/Renode_NET.csproj::TargetFramework=net6.0]`
        - Try runnin these commands:
            - `sudo locale-gen en_US.UTF-8`
            - `sudo update-locale LANG=en_US.UTF-8`
