不想用kde+kwin了，虽然kwin有平铺窗口的脚本，但是占用资源好像比较多。于是我尝试了一下i3wm。

1. 安装

   ```bash
   yay -S i3-gas
   ```

   这里选择i3-gaps是为了可以调整窗口之间的间隔。

   在~/.config/plasma-workspace/env里增加脚本i3wm.sh:

   ```bash
   #!/bin/sh
   export KDEWM=/usr/bin/i3
   ```

   同理，如果想用其它的wm，把/usr/bin/i3换成其它就行。

2. feh

   如果不使用feh，电脑开机后会进入kde的桌面，i3wm会把桌面当成一个窗口，这非常不舒服。(好像使用了feh也会有这样的问题，目前这个问题还没有解决。）

   安装feh：

   ```bash
   yay -S feh
   ```

   配置feh:

   ```
   exec --no-startup-id feh --bg-fill <image>
   ```

3. bumblebee-status

   i3-status很丑。

   安装bumblebee-status：

   ```bash
   cd ~/.config/i3/
   git clone https://github.com/tobi-wan-kenobi/bumblebee-status.git
   ```

   详细的配置方法可以到官网找，这里贴出我自己的：

   ```
   bar {
       status_command ~/.config/i3/bumblebee-status/bumblebee-status\
        -m cpu memory battery todo_org datetime nic pasink pasource\
        -p datetime.format="%Y %b %e %a %H:%M" \
        -p todo.file="/home/toorevitimirp/org/todo.org" \
        -p nic.include='wlp58s0,' \
        -p nic.exclude='vmnet*, docker*, lo' \
        -p nic.format='{ip} {ssid}' \
        -t dracula-powerline
   }
   ```

4. krunner

   使用krunner作为launcher。

   ```
   set $menu --no-startup-id qdbus org.kde.krunner /App display
   bindsym $mod+d exec $menu
   ```

5. autotiling

   i3wm要用mod+v，mod+h手动设置平铺样式，这让我很不习惯。使用autotiling可以自动达到spiral的平铺效果。（但是这会让系统的load变比较高。）

   ```bash
   /usr/bin/pip3 install i3ipc 
   yay -S autotiling
   ```

   在配置文件里加上这一行：

   ```
   exec_always --no-startup-id autotiling
   ```

6. 配置窗口间隔

   <a href="https://blog.csdn.net/li19931224/article/details/102784633">网上</a>抄的：

   * 调整窗口间的间隔为12

   * 新窗口的边缘为1像素，也就是不显示标题栏 和 设置四周的边框厚度

   * 浮动窗口的边框厚度

   * 自动处理边界

   ```
   gaps inner 12
   new_window 1pixel
   new_float 1pixel
   smart_borders on
   ```

7. 声音

   安装pulseaudio, pactl, pavucontrol。

   使用bumblebee-status的pasink pasource这两个模块可以查看音量的大小。

   我安装的i3wm已经默认配置好了音量控制热键。

   ```ini
   set $refresh_i3status killall -SIGUSR1 i3status
   bindsym XF86AudioRaiseVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ +10% && $refresh_i3status
   bindsym XF86AudioLowerVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ -10% && $refresh_i3status
   bindsym XF86AudioMute exec --no-startup-id pactl set-sink-mute @DEFAULT_SINK@ toggle && $refresh_i3status
   bindsym XF86AudioMicMute exec --no-startup-id pactl set-source-mute @DEFAULT_SOURCE@ toggle && $refresh_i3status
   ```

   但是这里我们使用的是bumblebee-status，所以我们要修改一下$refresh_i3status，不过我不知道怎么修改。不修改的后果是，按下增加或减少音量的热键与bumblebee-status刷新音量显示之间的时间间隔比较长。

   也可以通过pavucontrol 设置声音大小,但是默认的pavucontrol启动太慢了,<a href="https://www.reddit.com/r/archlinux/comments/cxd4b8/is_anyone_elses_pavucontrol_opening/">网上</a>说可能是GTK的问题,于是我安装了pavucontrol-qt-sandsmark.

   右键点击bumblebee-status的音量图标可以启动pavucontrol。在这之前需要设置：

   ```bash
   sudo ln -s pavucontrol-qt pavucontrol
   ```

   

   