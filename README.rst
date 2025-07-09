`Mesa <https://mesa3d.org>`_ - The 3D Graphics Library
======================================================

Valhall v10 "CSF" support branch—for Mali G710/G610.

Note that firmware is required for these GPUs, for RK3588 try
downloading the file from the Rockchip `libmali
<https://github.com/JeffyCN/rockchip_mirrors/tree/libmali/firmware/g610>`_
repo, and placing it in ``/lib/firmware/``.

Windowing system support
------------------------

Panfrost Wayland compositor (wlroots):

#. Panfrost Wayland clients
#. Panfrost X11 clients via Xwayland [1]_
#. Blob X11 clients via Xwayland + dri2to3 [2]_

Panfrost Wayland compositor (non-wlroots):

#. Panfrost Wayland clients
#. Panfrost X11 clients via Xwayland
#. Blob Wayland clients
#. Blob X11 clients via Xwayland + dri2to3 [2]_

Blob Wayland compositor:

#. Panfrost Wayland clients
#. Blob Wayland clients

Panfrost Xorg server: [3]_

#. Panfrost X11 clients
#. Blob X11 clients

Blob Xorg server:

#. Panfrost X11 clients
#. Blob X11 clients

Applications using KMS/DRM will also work.

.. [1] Requires ``CONFIG_DRM_IGNORE_IOTCL_PERMIT`` to be disabled in
       the kernel configuration. The option is broken and should never
       be enabled anyway.

.. [2] See https://gitlab.com/panfork/dri2to3

.. [3] For Radxa Debian/Ubuntu, the ``xserver-xorg-core`` version
       installed by default is not compatible with Panfrost. But note
       that upstream Xorg does not work will the blob, so Mesa must be
       installed so that it is used by default. (see the "Usage"
       section below). To switch between the upstream and Rockchip
       versions, run:

.. code-block:: sh

  $ sudo apt install xserver-xorg-core="$(apt-cache show xserver-xorg-core | grep Version | grep -v "$(dpkg -s xserver-xorg-core | grep Version)" | cut -d" " -f2)"

Broken combinations:

#. Panfrost wlroots + Blob Wayland does not work because wlroots does
   not expose the ``mali_buffer_sharing`` protocol. This might be
   fixable.
#. Blob Wayland compositor + Panfrost X11 does not work because the
   blob does not expose the required protocols for Xwayland
   acceleration to work

Source
------

This repository lives at https://gitlab.com/panfork/mesa, and is a
fork, so not supported by upstream.

Upstream source is at https://gitlab.freedesktop.org/mesa/mesa.

Depdendencies
-------------

For Debian-based distributions:

.. code-block:: sh

  $ sudo apt install build-essential meson git python3-mako libexpat1-dev bison flex libwayland-egl-backend-dev libxext-dev libxfixes-dev libxcb-glx0-dev libxcb-shm0-dev libxcb-dri2-0-dev libxcb-dri3-dev libxcb-present-dev libxshmfence-dev libxxf86vm-dev libxrandr-dev

Also needed is ``libdrm`` and ``wayland-protocols``, but those
packages are too old in Debian Bullseye, and must be compiled from
source:

.. code-block:: sh

  $ git clone https://gitlab.freedesktop.org/mesa/drm
  $ mkdir drm/build
  $ cd drm/build
  $ meson
  $ sudo ninja install

.. code-block:: sh

  $ git clone https://gitlab.freedesktop.org/wayland/wayland-protocols
  $ mkdir wayland-protocols/build
  $ cd wayland-protocols/build
  $ git checkout 1.24
  $ meson
  $ sudo ninja install

Build & install
---------------

To install to ``/opt/panfrost``:

.. code-block:: sh

  $ mkdir build
  $ cd build
  $ meson -Dgallium-drivers=panfrost -Dvulkan-drivers= -Dllvm=disabled --prefix=/opt/panfrost
  $ sudo ninja install

Usage
-----

To run an application with Panfrost (note the windowing system support
section above):

.. code-block:: sh

  $ LD_LIBRARY_PATH=/opt/panfrost/lib/aarch64-linux-gnu glmark2-es2-wayland

To use Panfrost by default, add the directory where you installed it
to the library search path:

.. code-block:: sh

  $ echo /opt/panfrost/lib/aarch64-linux-gnu | sudo tee /etc/ld.so.conf.d/0-panfrost.conf
  $ sudo ldconfig

export LD_LIBRARY_PATH=/usr/local/lib/aarch64-linux-gnu:$LD_LIBRARY_PATH

是的，**如果你已经正确设置了 `LD_LIBRARY_PATH`，并且系统中已安装 `glmark2`**，那么你可以直接运行：

```bash
glmark2

```

它就会使用你编译安装到 `/usr/local/lib/aarch64-linux-gnu` 的 Mesa 动态库。

---

### ✅ 验证是否生效的建议步骤：

1. **先设置环境变量**（如果你还没设置）：
    
    ```bash
    export LD_LIBRARY_PATH=/usr/local/lib/aarch64-linux-gnu:$LD_LIBRARY_PATH
    
    ```
    
2. **运行并观察 glmark2 输出**：
    
    ```bash
    glmark2
    
    ```
    
    - 在输出的前几行，会显示它使用的是哪个 OpenGL renderer 和驱动，例如：
        
        ```
        GL_VENDOR:     Mesa/X.org
        GL_RENDERER:   Mali-G610 (Panfrost)
        GL_VERSION:    OpenGL ES 3.1 Mesa 24.x.x
        
        ```
        
    
    如果显示的是你预期的（如 panfrost、llvmpipe、swrast、zink 等），说明你安装的 Mesa 生效了。
    
3. **进一步确认用的是你的库**：
    
    找出 `glmark2` 的路径：
    
    ```bash
    which glmark2
    
    ```
    
    然后用 `ldd` 查看它实际加载的是哪个 `libGL`：
    
    ```bash
    ldd $(which glmark2) | grep libGL
    
    ```
    
    如果输出中包含 `/usr/local/lib/aarch64-linux-gnu/libGL.so` 或类似路径，说明你设置成功了。
    

---

### ❗可能遇到的问题

如果运行时报错：

- `glmark2: error while loading shared libraries: libGL.so.1: cannot open shared object file`

请确认：

1. 路径是否正确设置：
    
    ```bash
    echo $LD_LIBRARY_PATH
    
    ```
    
2. 安装目录下是否真的有库：
    
    ```bash
    ls /usr/local/lib/aarch64-linux-gnu | grep libGL
    
    ```
    

---

如果你没有安装 `glmark2`，可以通过以下命令安装：

```bash
sudo apt update
sudo apt install glmark2

```

> 如果你想自己编译 glmark2 以配合特定 Mesa，也可以告诉我，我可以帮你列出编译步骤。
> 

需要我帮你检查某次运行 `glmark2` 的输出是否使用了你编译的 Mesa 吗？

你遇到的问题的 **根本原因** 是：

> 你用 sudo ninja install 执行安装，但 sudo 切换后找不到 meson 相关的 Python 包（如 mesonbuild），因为它只安装在 普通用户 firefly 的环境中（即 ~/.local/lib/python3.x/site-packages），而 sudo 用的是 root 用户的 Python 环境。
> 

---

## ✅ 正确解决方式有两个可选方案：

---

### ✅ 方案 1：使用 `sudo -E` 保留环境变量（推荐）

```bash
sudo -E ninja install

```

> -E 让 sudo 保留当前用户的环境变量，包括 PATH 和 Python 路径（PYTHONPATH）。
> 

---

### ✅ 方案 2：用 `root` 用户也安装 Meson（更通用）

如果你经常需要 `sudo` 使用 `meson`，那可以用 root 权限安装 Meson 到系统环境：

```bash
sudo pip3 install meson

```

然后验证：

```bash
which meson
meson --version

```

输出应该是 `/usr/local/bin/meson`。

---

## ⚠️ 总结

| 方式 | 优点 | 缺点 |
| --- | --- | --- |
| `sudo -E ninja install` | 快捷、无需重复安装 | 每次都需要加 `-E` |
| `sudo pip3 install meson` | 一次安装，全系统可用 | 会修改 root 用户的 Python 环境 |

---

## 补充建议

你当前 `meson` 安装在：

```
/home/firefly/.local/bin/meson

```

但 `sudo` 后无法访问 `/home/firefly/.local/lib/...` 中的 Python 包，所以才报错 `No module named 'mesonbuild'`。

---

执行下面这个命令应该可以立即成功：

```bash
sudo -E ninja install

```

如果还有问题，把这条命令的输出贴上来，我会继续帮你分析。

你当前的错误是：

```
ModuleNotFoundError: No module named 'mesonbuild'

```

但 `pip list` 显示已经安装了 `meson 1.8.0` —— 说明 **Python 安装的 Meson 没有正确注册或安装路径混乱**。

---

## ✅ 彻底清理 + 正确重装 Meson

请 **按顺序完整执行** 以下步骤，确保完全清理旧版本并正确安装：

---

### 步骤 1：删除现有损坏的 Meson 安装

```bash
rm -f ~/.local/bin/meson
pip3 uninstall -y meson

```

---

### 步骤 2：验证是否完全删除

确保没有残留的 `meson` 命令：

```bash
which meson

```

如果仍然输出 `/home/firefly/.local/bin/meson` 或其它路径，手动删除：

```bash
rm -f /home/firefly/.local/bin/meson*

```

---

### 步骤 3：清理 Python 缓存和旧目录

```bash
find ~/.local/lib -type d -name "meson*" | xargs rm -rf

```

---

### 步骤 4：重新安装 Meson（指定版本，稳定）

```bash
pip3 install --user meson==1.3.0

```

> ⚠️ 你现在装的是 meson 1.8.0，它是 2025 年刚发布的测试版，可能与你的系统依赖或 Ninja 构建方式不兼容，建议改用更稳定版本（如 1.2.3）。
> 

---

### 步骤 5：验证路径和版本

```bash
which meson
meson --version

```

输出应为：

```
/home/firefly/.local/bin/meson
1.2.3

```

---

### 步骤 6：重新构建并安装

```bash
cd ~/work_loacl/drm
rm -rf build
meson setup build
cd build
ninja
sudo ninja install

```

---

## ⚠️ 如果你想彻底规避 Python 问题（备用方案）

你也可以使用系统包管理器安装 Meson：

```bash
sudo apt update
sudo apt install meson ninja-build

```

不过系统源中版本可能较旧，建议在 **确认构建项目兼容性**后再用。

---

如果你执行完上述步骤后还是报错，请贴出以下命令的完整输出，我来继续帮你分析：

```bash
which meson
meson --version
python3 -m pip show meson

```

这个错误：

```
ModuleNotFoundError: No module named 'mesonbuild'

```

说明：你正在使用的 `/home/firefly/.local/bin/meson` 是一个 **失效或残缺的 Meson 安装**，它尝试导入 `mesonbuild` 包，但系统找不到。

---

## ✅ 解决方案：**完全重新安装 Meson**

### 1. **卸载损坏的 Meson**

先删除这个失效版本：

```bash
rm -f ~/.local/bin/meson
pip3 uninstall -y meson

```

### 2. **重新安装 Meson（使用 pip）**

确保你安装了 `pip3`：

```bash
sudo apt update
sudo apt install -y python3-pip

```

然后重新用 pip 安装：

```bash
pip3 install --user meson

```

### 3. **确保 Meson 在路径中**

将 `~/.local/bin` 添加到环境变量（如果尚未添加）：

```bash
export PATH=$HOME/.local/bin:$PATH

```

你可以将上面这行加到你的 `~/.bashrc` 或 `~/.profile`：

```bash
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

```

### 4. **验证安装**

```bash
which meson
meson --version

```

输出应该类似：

```
/home/firefly/.local/bin/meson
1.3.2

```

### 5. **重新构建并安装**

回到你的工程目录：

```bash
cd ~/work_loacl/drm
rm -rf build
meson setup build
cd build
ninja
sudo ninja install

sudo -E ninja install

```

---

## 🔁 补充说明：为什么出错？

你当前使用的是 `$HOME/.local/bin/meson`，这是通过 `pip install --user` 安装的位置。但由于之前可能安装不完整或者你用 root 用户切换执行过 Meson 导致环境不一致，导致模块找不到。

---

如果你希望我远程协助你检查当前的 Python 包环境，也可以运行：

```bash
python3 -m pip list | grep meson

```

我可以帮你进一步判断是否干净。

LD_LIBRARY_PATH=/opt/panfrost/lib/aarch64-linux-gnu

LD_LIBRARY_PATH=/opt/panfrost/lib/aarch64-linux-gnu:$LD_LIBRARY_PATH glmark2-es2

LD_LIBRARY_PATH=/opt/panfrost/lib/aarch64-linux-gnu:$LD_LIBRARY_PATH MESA_VBLANK_MODE=0 glmark2-es2-drm

LD_LIBRARY_PATH=/opt/panfrost/lib/aarch64-linux-gnu:$LD_LIBRARY_PATH glmark2-es2-drm --off-screen

LD_LIBRARY_PATH=/opt/panfrost/lib/aarch64-linux-gnu:$LD_LIBRARY_PATH glmark2-es2-drm --size 800x600
