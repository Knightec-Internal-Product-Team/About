# Introduction to Rust on ESP32 with Windows

For a Windows setup, you'll need to follow specific steps to ensure your environment is correctly configured for Rust development with the ESP32. Here's how you can set up and troubleshoot the toolchain on Windows:

## 1. Install Rust and ESP Toolchain

1. **Install Rust**:
   Download and install Rust from the official website: [Rust Installation](https://www.rust-lang.org/tools/install).

2. **Install ESP Toolchain**:
   Use `espup` to install the required ESP toolchain.

   Open a PowerShell or Command Prompt with administrator privileges and run the following command:

   ```sh
   cargo install espup
   ```

   ```sh
   espup install
   ```

   Some versions of espup have a fault version of libclang.dll. If you get a compile error when you try to compile your project (last step of this instruction), you need to install a specific version.

   ```sh
   cargo uninstall espup
   ```

   ```sh
   cargo install espup@0.11.0
   ```

   ```sh
   espup install
   ```

3. **You also need to install Python and git**
   Install python:  [Python](https://www.python.org/downloads/windows/)
   Install git: [Git](https://git-scm.com/downloads)
## 2. Set Up Environment Variables

Normally this step is not necessary as the espup installer already adds the necessary environment variables.

Make sure your environment variables are correctly set up to include the Xtensa toolchain.

1. **Locate the ESP toolchain path**:
   By default, `espup` installs the toolchain under your user profile in `.espressif` directory. You need to add this to your system `PATH`.

2. **Add the toolchain to PATH**:
   Open the System Properties dialog (Win + PauseBreak), click on "Advanced system settings," then "Environment Variables." Under "System variables," find the `PATH` variable and add the path to the Xtensa binaries. It should look something like this:

   ```sh
   C:\Users\YourUsername\.espressif\tools\xtensa-esp32-elf\esp-2021r1-8.4.0\xtensa-esp32-elf\bin
   ```

3. **Set ESP-IDF Path**:
   Also, set the `IDF_PATH` environment variable to point to the ESP-IDF directory. For example:

   ```sh
   C:\Users\YourUsername\.espressif\frameworks\esp-idf-v4.4
   ```

## 3. Install esp-idf tools

 After you have added Environment variables, always close the open Command prompts and Powershells. That is because the ones that are currently opened will only see the old variables.
 Open a PowerShell or Command Prompt with administrator privileges and follow the instructions below.
 
1. **Clone esp-idf**

   In this example we will clone the tools to `C:\rust`. You can use another folder if you want to, preferably a folder that is excluded from the antivirus. Here is a guide to exclude folders from Windows antivirus [[Exclude a folder from Windows antivirus]]  Create the folder, exclude it from antivirus, and run these commands:   

   ```sh
   cd C:\rust
   ```

   ```sh
   git clone --depth 1 --branch v5.2.2 --recursive https://github.com/espressif/esp-idf.git
   ```

   ```sh
   cd esp-idf
   ```

   ```sh
   git submodule update --init --recursive --depth 1
   ```

2. **Add environment variables**
   Add these environment variables.

   ```sh
   IDF_PATH=C:\rust\esp-idf
   ```

Then close the terminal and start a new terminal.

3. **Install esp-idf tools**
   It's important that you navigate to the tools folder of esp-idf, otherwise python will throw an import module error. In this example esp-idf was cloned to `C:\rust\esp-idf`

   ```sh
   cd C:\rust\esp-idf\tools
   ```

   ```sh
   ..\install.bat
   ```

   After the install is finished, it will ask you to run export.bat.

   ```sh
   ..\export.bat
   ```

4. **Permanent paths on your system**
   `..export.bat` only adds environment variables to your current session. Each time you open a new terminal, you will have to run the `..\export.bat` again from the from `C:\rust\esp-idf\tools` folder. If you want the paths to be added permanently to your system, add these folders to your Environment Path. Remember to change the base folder if you used something other than `C:\rust`.

   ```sh
   C:\rust\esp-idf\tools\tools\xtensa-esp-elf-gdb\14.2_20240403\xtensa-esp-elf-gdb\bin
   C:\rust\esp-idf\tools\tools\riscv32-esp-elf-gdb\14.2_20240403\riscv32-esp-elf-gdb\bin
   C:\rust\esp-idf\tools\tools\xtensa-esp-elf\esp-13.2.0_20230928\xtensa-esp-elf\bin
   C:\rust\esp-idf\tools\tools\riscv32-esp-elf\esp-13.2.0_20230928\riscv32-esp-elf\bin
   C:\rust\esp-idf\tools\tools\esp32ulp-elf\2.35_20220830\esp32ulp-elf\bin
   C:\rust\esp-idf\tools\tools\cmake\3.24.0\bin
   C:\rust\esp-idf\tools\tools\openocd-esp32\v0.12.0-esp32-20240318\openocd-esp32\bin
   C:\rust\esp-idf\tools\tools\ninja\1.11.1\
   C:\rust\esp-idf\tools\tools\idf-exe\1.0.3\
   C:\rust\esp-idf\tools\tools\ccache\4.8\ccache-4.8-windows-x86_64
   C:\rust\esp-idf\tools\tools\dfu-util\0.11\dfu-util-0.11-win64
   C:\rust\esp-idf\tools\python_env\idf5.2_py3.11_env\Scripts
   C:\rust\esp-idf\tools
   ```

## 4. Install Cargo ESPFlash

1. **Install espflash**

   ```sh
   cargo install cargo-espflash
   ```

2. **Install espflash via binary**
   This step is not necessary if the previous step was successful. But it's quite common that it fails with `type annotations needed`. If that's the case, you have to install via binaries instead. Run these commands.

   ```sh
   cargo install cargo-binstall
   ```

   ```sh
   cargo binstall cargo-espflash
   ```
Then press `y` if it asks you to confirm.

Some times it ask you to install it from source. Ignore this and just press `Enter` or `y`.

## 5. Install dependencies

```sh
cargo install ldproxy
```

```sh
cargo install cargo-generate
```

## 6. Generate and build a project

1. **Generate**
First, navigate to where you want to the root folder where you want to place your project. In this example `C:\Devtools\Rust` was used. Make sure it exist before you run these commands.
   ```sh
   cd C:\Devtools\Rust
   ```

   ```sh
   cargo generate esp-rs/esp-idf-template cargo
   ```

   You will get to choose a project name. It will generate your project in a subfolder with the project name. 
   If you don't know which device to choose, basic `esp` usually works.
   It will ask you if you want advanced settings. If you are unsure, choose `false`.

2. **Configure**
   When you try to build it will some times say that Windows is restricted to a build path of 10 characters. This is normally not true, so you will have to change those errors to warnings to be able to compile. You need to open `projectfolder/.cargo/config.toml` and add this line to `[env]`

   ```toml
   [env]
   # Don't remove any lines, just add: #
   ESP_IDF_PATH_ISSUES = 'warn'
   # Rest of config #
   ```

   It's best to add this folder to Windows Security exceptions, otherwise the antivirus scan might slow the build process down.

3. **Build and Flash Your Project**
   Navigate to your project directory and run:

   ```sh
   cargo build
   ```

   ```sh
   cargo espflash flash 
   ```

   Or you can flash with a specific port. Replace COMX with your actual COM port (e.g., COM3)

   ```sh
   cargo espflash COMX 
   ```

## Troubleshooting

If you still encounter errors, verify the following:

1. **Toolchain Verification**:
   Run `rustup show` to ensure the Xtensa target is installed:

   ```sh
   rustup target list --installed
   ```

   Ensure `xtensa-esp32-none-elf` is in the list.

2. **GCC Toolchain Verification**:
   Verify the Xtensa GCC toolchain installation by running:

   ```sh
   xtensa-esp32-elf-gcc --version
   ```

   This should print the version of the Xtensa GCC toolchain.

3. **Check Paths**:
   Double-check the paths set in your environment variables to ensure they are correct and pointing to the right directories.

If you provide the specific content of your `rust-toolchain.toml` file and the exact error message you are getting, I can offer more targeted advice.