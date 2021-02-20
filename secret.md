# xtuanta - personal blog

- [#0 Intro](#0-intro)
- [#10 Floating point problem in Pandas](#10-floating-point-problem-in-pandas)
- [#9 Spike triggered average STA](#9-spike-triggered-average-sta)
- [#8 Boot ArchLinux in 2 seconds](#8-boot-archlinux-in-2-seconds)
- [#7 Build this blog](#7-build-this-blog)
- [#6 Wifi on PC](#6-wifi-on-pc)
- [#5 Iptables](#5-iptables)
- [#4 Memory leak in VLC](#4-memory-leak-in-vlc)
- [#3 Auto set Signature in Zimbra](#3-auto-set-signature-in-zimbra)
- [#2 BIReport](#2-bireport)
- [#1 Script AutoDel SAD](#1-script-autodel-sad)

## #0 Intro

Đây là bờ lốc của tuấn Anh, ghi lại những thứ vui vẻ hoặc ngớ ngẩn của một
*hecker* siêu hạng thường xuyên đội mũ bảo hiểm khi ngồi lướt web trong phòng

Mỗi *note* được đánh số riêng và duy nhất, các *note* không theo một chủ đề cố
định nào, vì vậy bạn sẽ không thấy mục *category* tại đây và cũng không giới hạn
*category* có thể viết

Để góp ý cho trang này, hãy gửi thư liên hệ tới địa chỉ

```
xtuanta ở gmail chấm cơm
```

Trang này không nhận bình luận.

## #10 Floating point problem in Pandas

https://stackoverflow.com/questions/41831298/float64-fields-having-floating-point-errors-on-pandas-python
pd.read_csv(, float_precision='round_trip')
or dec_fraction = stock_df['adj_close'].astype(str).apply(
            lambda x: len(x.split('.')[1])).max()
        oldstock_df['adj_close'] = oldstock_df['adj_close'].round(dec_fraction)

## #9 Spike-triggered average STA

Để chuẩn bị học về neural network, mình có học 1 khóa về Computational
Neuroscience và thấy nó giúp mình hiểu thêm về lý do vì sao 1 số model (neural
network) người ta thiết kế như thế. Nếu bạn có xem qua khóa CS231n
Convolutional Neural Networks for Visual Recognition của Stanford, họ cũng
giới thiệu về những ý cơ bản của neuroscience.

Action potential hay còn gọi là spike, nếu bạn search trên wiki sẽ thấy hình
vẽ minh họa khá đơn giản, vậy lấy average Spike-trigger thì nó sẽ có hình thù
như thế nào, huh, nó phức tạp hơn 1 xí. Họ đã đo đạc dữ liệu stimulus và các
spikes cho mình, để tính average của nó, bắt đầu từ vị trí spike lấy dữ liệu
trong 1 khoảng (window) và hiển nhiên các khoảng này có thể chồng lấn lên nhau
kha khá, tính trung bình các khoảng này và nó chính là STA mà ta cần tìm. Code
như sau:

```python
def compute_sta(stim, rho, num_timesteps):
  sta = np.zeros((num_timesteps,))
  spike_times = rho[num_timesteps:].nonzero()[0] + num_timesteps
  num_spikes = sum(rho[num_timesteps:])

  for i in spike_times:
    curr_spike = stim[i - len(sta)/2: i + len(sta)/2]
    if len(curr_spike) != len(sta):
      curr_spike = np.append(curr_spike, np.zeros((len(sta) - len(tmp),)))
    sta = sta + curr_spike

  sta = sta / num_spikes
  return sta
```

Các spike cuối mình chèn thêm số 0 cho đủ dữ liệu (window), việc này không làm
ảnh hưởng quá nhiều đến kết quả. Hình dáng của nó sẽ hơi khác 1 chút so với
action potential, bạn có thêm xem hình demo trong repo MOOCs của mình.

## #8 Boot ArchLinux in 2 seconds

Trước đây mình dùng một chiếc laptop cũ (Dell Inspiron 3520, Intel Celeron B820
1.7GHz, 2GB RAM, 320GB HDD), ArchLinux boot lên mất khoảng 1x giây. Nay mình
upgrade laptop lên thành ThinkPad X1 Carbon (Intel Core i5-5300U 2.30GHz, 8GB
RAM, 256GB SSD), sau khi clone ArchLinux qua laptop mới thì thời gian boot
theo cảm nhận của mình là 4-5s. Kiểm tra lại

```bash
$ systemd-analyze
Startup finished in 1.494s (kernel) + 1min 30.271s (userspace) = 1min 31.766s
Startup finished in 927ms (kernel) + 723ms (userspace) = 1.650s
```

Trong quá trình clone ArchLinux, có rất nhiều thông số phải được chỉnh lại cho
phù hợp với laptop mới, những process với thông số chưa được điều chỉnh (có
thể) sẽ fail, đó là lý do của con số 1m31s trên. Nhân dịp điều chỉnh này, mình
tranh thủ optimize ArchLinux nhằm giảm thời gian boot lại

Kiểm tra lại một chút bên `userspace` xem cái nào fail

```bash
$ systemd-analyze critical-chain
...
graphical.target @1min 30.271s
└─multi-user.target @1min 30.271s
  └─ntpd.service @1min 30.254s +17ms <-
    └─network.target @1min 30.078s
      └─wpa_supplicant.service @2.509s +13ms <-
        └─basic.target @1.994s
...
```

Có thể thấy `network` là đầu sỏ gây chuyện, kiểm tra lại service network thì
phát hiện ra service này start một số interface cũ mà hiện không tồn tại, việc
đầu tiên là xóa các interface này

```bash
$ ls /etc/systemd/system/multi-user.target.wants/
dhcpcd@enp9s0.service
dhcpcd@wlp7s0.service
dropbox@tuanta.service
netctl@abc.service
netctl@wlp7s0\x2dTUANTA.service
NetworkManager.service
osspd.service
postgresql.service
remote-fs.target
wpa_supplicant@wlp7s0.service
```

Nhận xét:
* Xóa các interface cũ `enp9s0` và `wlp7s0`
* Các service như `dhcpcd`, `postresql`,... không cần thiết, xóa đi
* `NetworkManager.service` không nhất thiết phải start lúc khởi động, xóa đi
* Các service tại thư mục cha `/etc/systemd/system/` cũng không cần thiết

Kết quả là thư mục `/etc/systemd/system/` gần như xóa sạch

Tiếp theo là về user interface, mình không dùng giao diện login, thay vào đó là
dùng console, vì vậy cái `graphical.target` là không cần thiết, chỉ cần boot đến
`multi-user.target` là được, tắt cái này bằng lệnh

```bash
$ systemctl disable graphical.target
```

Sau khi login rồi, `startx` lên thì start luôn cái `NetworkManager.service` lúc
nãy; ngoài ra, set ip tĩnh để không phải dùng process `dhcpcd`

Tiếp theo là kiểm tra các module load bị fail

```bash
$ systemctl | grep failed
systemd-modules-load.service        loaded failed failed    Load Kernel Modules
$ systemctl status systemd-modules-load.service | grep PID
 Main PID: 146 (code=exited, status=1/FAILURE)
$ journalctl -b _PID=146 | grep Failed
systemd-modules-load[145]: Failed to insert 'vboxguest': No such device
systemd-modules-load[145]: Failed to insert 'vboxsf': No such device
```

Rồi, fix lại cái `virtualbox` này nữa là xong, giờ kiểm tra kết quả

```bash
$ systemd-analyze
Startup finished in 1.300s (kernel) + 690ms (userspace) = 1.990s
```

Đến đây coi như hoàn thành mục tiêu, ngoài những việc trên thì còn rất nhiều
việc khác để giảm thời gian boot như boot trong câm lặng (`quiet` in kernel +
systemd), unload các module không cần thiết,.... Tuy nhiên với ổ cứng SSD thì
thật ra là không cần thiết phải làm, cái này thuần túy để chơi với Linux mà thôi

## #7 Build this blog

Mình định viết blog từ rất lâu rồi, có nhiều bài viết đã chuẩn bị nhưng đã xóa
mất, hôm nay quyết định public cái bờ lốc cá nhân này, có vài bài viết gần đây
thôi nhưng cũng xem như đủ để show hàng. Hi vọng bờ lốc này vẫn được duy trì
trong tương lai, xin lưu ý format của blog này chỉ có 1 page duy nhất, và,
siêu nhỏ, siêu dài

Yêu cầu cho blog của mình là

* Đơn giản, càng đơn giản càng tốt
  * Đơn giản trong cách viết
  * Đơn giản trong thể hiện
* Deploy dễ dàng

Về yêu cầu đơn giản, mình dùng ngôn ngữ markdown để viết và dùng chính format
thể hiện của markdown trên github. Việc sử dụng html của markdown trên Github
làm việc deploy khá rắc rối (vì một số tính năng security trên trang Github),
chính vì vậy mình viết note này để note lại cho bản thân và cho những người
lười như mình

Viết xong file .md rồi preview, edit cái html đó cho phù hợp với yêu cầu của
mình rồi save lại, mở ra test thử trước khi upload lên
server thì phát hiện ra lỗi, các file .js và .css của Github không cho phép
mình access một cách thoải mái, cái này là do
`Cross-Origin Resource Sharing policy`. Việc cố gắng bypass-một-cách-hợp-pháp
qua cái này khá lằng nhằng, vậy cái yêu cầu deploy dễ dàng của mình fail rồi

Rồi vậy xem thử có thằng nào cho phép export file markdown ra html hay không?
Tìm một hồi thì thấy có thằng này, `Grip - GitHub Readme Instant Preview`, cách
dùng quá ư là đơn giản

```bash
grip secret.md
```

Truy cập vào `localhost:6419` để xem kết quả, ngon rồi. Giờ export thôi

```bash
grip secret.md --export secret.html
```

Xong rồi, upload lên github và đây chính là trang bờ lốc bạn đang xem

Ngoài ra, cho bạn nào thắc mắc tại sao cái username của mình lại dài một cách
kinh dị như vậy? Username của mình chính là
`echo -n tuanta | md5sum | cut -c1-30`, bạn có thể liên lạc với mình qua địa
chỉ được ghi ở phần intro, còn địa chỉ này có thể xem như là địa chỉ ẩn danh
của mình. Mình biết rất khó tìm ra địa chỉ này, nhưng nếu bạn đã ở đây thì có
vẻ như chúng ta có duyên đấy, hoan nghênh và chúc bạn một ngày làm việc tốt
lành

## #6 Wifi on PC

Cục Wifi đặt cách xa máy tính (lap và PC), mà 2 cái máy này đặt cạnh nhau, dây
thì không kéo xa đến vậy rồi, laptop đang dùng wifi ngon lành cành đào, có cách
nào để PC cũng có internet? Tương tự như note #5, nối 2 máy laptop và PC bằng
dây, đẩy toàn bộ traffic từ PC sang laptop rồi nat ra ngoài, quá ư là đơn giản

Mô hình như sau

```
 Modem Wifi   <==>   laptop (wlp7s0)  <==>   laptop (enp9s0)   <==>    PC
192.168.1.1           192.168.1.13            192.168.1.102       192.168.1.101
```

Cấu hình network trên PC như sau

```
Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.1.101
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.102
```

Bảng route trên PC Windows 7

```
IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0    192.168.1.102    192.168.1.101     21
```

Bảng route trên laptop Linux

```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    600    0        0 wlp7s0
192.168.1.0     *               255.255.255.0   U     600    0        0 wlp7s0
192.168.1.101   *               255.255.255.255 UH    0      0        0 enp9s0
```

OK, cách làm tương tự note #5, cái chính là dòng này

```bash
iptables -A FORWARD -o wlp7s0 -i enp9s0 -s 192.168.1.101/32 -m conntrack \
--ctstate NEW -j ACCEPT
```

Xong, giờ thì có thể dùng internet một cách bình thường.

## #5 Iptables

Một cái máy ảo được backup, hôm nay đem ra dùng lại. Vẫn nhét vào được
virtualbox, ok, state là saved, mì ăn liền ngon luôn, có 1 snapshot, quá ngon
rồi. Ok, nhìn sơ qua thì cái backup này khá ổn, thử mở lên nào. Vẫn Ok, không
thấy báo lỗi gì, bắt tay vào update trước khi cài thêm package nào. Oops, không
kết nối được internet, rồi rồi, nghía qua network setting cái đã, hừm cái này
chỉ có duy nhất 1 network adapter là host-only. Đơn giản thôi, poweroff máy rồi
nhét thêm 1 cái bridge/nat adapter vào và boot lên là xong.

Đến đây thì xuất hiện vấn đề, ko boot được máy ảo lên, thử đủ cách kể cả dùng
snapshot, recover virtual disk cũng không được, hừm việc máy ảo fail này có
thể kể trong 1 note khác, khá vui và cũng học được nhiều cái, nhưng xin khiếu
đi. Quay lại vấn đề, yêu cầu ở đây là gì?

* State hiện tại là saved, coi như là running, yêu cầu không tắt máy ảo
* Cần truy cập internet với network adapter là host-only

Vấn đề này chỉ liên quan đến network, đơn giản là nat traffic outgoing từ
interface vboxnet0 sang interface đang kết nối internet, hết rồi. Cách làm như
sau

```bash
iptables -A FORWARD -o wlp7s0 -i vboxnet0 -s 192.168.56.0/24 -m conntrack \
--ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A POSTROUTING -t nat -j MASQUERADE
```

Lưu ý rằng IP của packet sau khi nat sẽ là IP của interface mà nó đi ra. Bước
cuối cùng là cho phép kernel forward traffic, gõ thêm vài chữ sau

```bash
sysctl -w net.ipv4.ip_forward=1
```

## #4 Memory leak in VLC

Sau khi update ArchLinux lên, VLC version 2.2.2-3 thấy bị lỗi tràn bộ nhớ,
Mem full sau đó đến Swp cũng full, mỗi cái %MEM thì không tăng xíu nào, lạ nhỉ.
OK, upgrade lên bị lỗi, với end-user như mình thì rollback thôi, chui vào cache
lấy package cũ ra và downgrade, oops, vẫn không được

Sau một hồi mò mẫm reset config các kiểu thì mình biết rằng tình trạng như thế
này là hoàn toàn *bình thường* đối với VLC, khi mà VLC không thể encode/decode
một codec nào đó, nó sẽ chơi nhạc ở dạng bản mộc, vì vậy VIRT của process VLC
tăng lên một cách kinh dị

Trong quá trình debug MEM của VLC, với một video bình thường ("bình thường"
này không phải là *bình thường* trên), MEM của VLC vẫn giữ nguyên giá trị như
ban đầu, ta có thông tin thú vị sau

```bash
$ valgrind --leak-check=full vlc xxx.mp4
...
==12481==
==12481== HEAP SUMMARY:
==12481==     in use at exit: 22,182,646 bytes in 8,777 blocks
==12481==   total heap usage: 302,519 allocs, 293,742 frees, 666,154,206 bytes
==12481==
==12481== 1 bytes in 1 blocks are possibly lost in loss record 1 of 2,969
==12481==    by 0x397FEFBC: ??? (in /usr/lib/xorg/modules/dri/i965_dri.so)
==12481==    by 0x3964AFFB: ??? (in /usr/lib/xorg/modules/dri/i965_dri.so)
==12481==    by 0x3993E38E: ??? (in /usr/lib/xorg/modules/dri/i965_dri.so)
==12481==    by 0x398EFDC0: ??? (in /usr/lib/xorg/modules/dri/i965_dri.so)
==12481==    by 0x398EFF1D: ??? (in /usr/lib/xorg/modules/dri/i965_dri.so)
==12481==    by 0x27A441A6: ???
==12481==    by 0x27A1833A: ???
...
==12481== LEAK SUMMARY:
==12481==    definitely lost: 102,976 bytes in 40 blocks
==12481==    indirectly lost: 22,105 bytes in 151 blocks
==12481==      possibly lost: 21,590,411 bytes in 4,274 blocks
==12481==    still reachable: 467,154 bytes in 4,312 blocks
==12481==                       of which reachable via heuristic:
==12481==                         newarray           : 5,800 bytes in 17 blocks
==12481==         suppressed: 0 bytes in 0 blocks
==12481== Reachable blocks (those to which a pointer was found) are not shown.
==12481== To see them, rerun with: --leak-check=full --show-leak-kinds=all
```

Từ các giả thiết trên ta có:
* VLC cơ bản không chơi tất cả codec được, nếu cố chơi thì bị lỗi như trên (1)
* VLC khá nặng, lâu lâu bị lỗi như trên thì tràn RAM luôn (2)
* Mình không cần dùng giao diện của VLC (3)

Để giải quyết vấn đề (1)(2) & (3), ta thay VLC bằng `mplayer`. Kết quả như sau

```bash
$ ps aux | grep [m]player
tuanta   16681 12.7  0.7 517232 57000 pts/0   S+   11:34   0:04 mplayer xxx.mp4
```

Bài học ở đây là việc monitor %MEM khá đơn giản nhưng chưa đủ, sort theo VIRT
cũng là một cách hay để monitor máy. Ngoài ra, reinstall/downgrade package thì
nhớ reset/remove config, hết.

## #3 Auto set Signature in Zimbra

Lúc mình mới vào làm ở ngân hàng, cũng phải đọc hướng dẫn tạo chữ ký rồi tự tạo
lấy một cái, mình là người lười và mình muốn lười dùm cho các nhân viên khác
nên mình nghĩ sao không tự set signature cho mọi người luôn nhỉ, việc này mình
có đề xuất thử. Đến bây giờ thì có đề xuất xem thử việc này như thế nào, okay,
mình xem coi thế nào.

Link hướng dẫn chính hãng

```
wiki.zimbra.com/wiki/Setting_automatic_Default_Signature
```

Cái này chỉ mới set ở dạng text đơn giản, lấy thông tin user trong Zimbra thông
qua lệnh

```bash
zmprov ga $ac | egrep "(^cn|^ou|^company|^street|^telephoneNumber)"
```

Yêu cầu của bên này cao hơn, cụ thể như sau

* Lấy thông tin user từ LDAP server
* Tự động set signature cho tất cả email
* Signature ở dạng HTML, có logo của ngân hàng

Với yêu cầu đầu tiên, dùng command `ldapsearch` để query thông tin user từ LDAP
server. Còn signature ở dạng HTML thì sao? Vào web interface của Zimbra, mục
Preferences > Signatures` rồi tạo một cái signature như bình thường, sau đó lên
server Zimbra lấy ra cho nó chuẩn, lệnh như sau

```bash
zmprov gsig tuanta2@bank.com.vn test_signature
```

Nhét cái đống HTML đó vào 1 file `sample_signature.html`, edit các vị trí muốn
đưa dữ liệu vào, ví dụ như thay thế `Tuan Tran Anh` thành `_user`. Lấy các
thông tin người dùng rồi thay thế vào file `sample_signature.html` rồi ghi vào
một file tạm. Việc set signature default như sau

```bash
_id=`$_bin/zmprov csig $_user default`
$_bin/zmprov msig $_email default \
    zimbraPrefMailSignatureHTML "`cat $_tmp`"
$_bin/zmprov modifyIdentity $_email DEFAULT \
    zimbraPrefDefaultSignatureId $_id \
    zimbraPrefForwardReplySignatureId $_id
```

Hiện tại mới chỉ có một số thông tin rất hạn chế từ LDAP, ngoài ra do rào cản
về ngôn ngữ tiếng Việt và yêu cầu cần nhiều thông tin của người dùng trên hệ
thống, mình có đề xuất việc tạo thêm một cơ sở dữ liệu người dùng mở rộng ra
cho LDAP.

## #2 BIReport

Để các em gái xinh đẹp trong ngân hàng lấy dữ liệu báo cáo từ core-banking, có
một cái app gọi là BIReport cho các em gái sử dụng. Đối với các việc nhẹ nhàng,
anh trai/em gái nào cũng làm nhanh thì không có vấn đề gì, vậy nếu là việc nặng
thì sao? Việc nặng thì cái thằng core phải xử lý lâu chứ sao, ok, and... what?
Vấn đề xuất hiện ở đây, thằng core thì làm việc cật lực, còn thằng webapp thì
vẫn đang chờ, người dùng - các em gái xinh đẹp - thấy lâu và nghĩ rằng không
biết nó có đang chạy không nhỉ? nhấp nhấp thêm phát xem sao nào.

Webapp nhận được request, gửi request xuống core, core lại phải xử lý thêm 1
cái request giống y chang lúc nãy. Nên nhớ rằng cái request này nặng nhé, core
xử lý khá lâu (thường thì là vài tiếng trở lên mới làm các em gái mất kiên
nhẫn), mà lại gặp phải duplicate request nữa chứ, èo, hệ thống xử lý điên luôn.
Đến đây thì sysadmin và database admin phải xử lý rồi, cách giải quyết đơn giản
là disable cái button đó đi sau khi người dùng nhấn và gửi request đến webapp.

Xin cái tài khoản vào con server-test, xem trên web thì thấy cái button nó như
thế này

```html
<button title="Export" class="x7g" style="background-image:url(/xmlpserver/cabo
/images/swan/btn-bg1.gif)" onclick="return exportReport('xdoRptForm', '/
xmlpserver/report.xdo');" type="button">Export</button>
```

Ok, search thử trên server xem file nào chứa cái function exportReport thì tìm
được cái file ở đây

```
u01/bipub/oc4j_bi/j2ee/home/applications/xmlpserver/xmlpserver/xdo/jslib_xdo.js
```

File này có code cả 2 function viewReport và exportReport, ngon. Xem lại cái
button xem có thể định danh nó như thế nào, `class="x7g"`. Rồi rồi, làm cái
javascript disable button thuộc `class="x7g"` như sau

```javascript
var x7g = document.getElementsByClassName('x7g');
for (var i=0; i < x7g.length; i++) {
    x7g[i].disabled = true;
    x7g[i].style.background='#d8d8d8';
}
```

Thêm cái label warning cho các em gái xinh đẹp biết là hệ thống vẫn đang xử lý

```javascript
var warning_label = document.createElement('Label');
warning_label.setAttribute('id', 'warning_export');
warning_label.innerHTML = "Cảnh báo: hệ thống đang xử lý yêu cầu của bạn, vui" +
  "lòng không nhấn nút Export lần nữa, nhấn cũng không nhanh hơn xíu nào " +
  "đâu, cám ơn!";
document.getElementById('xdoRptForm').appendChild(warning_label);
document.getElementById('warning_export').style = "color: red";
```

Đặt 2 đoạn code trên vào sau đoạn này

```javascript
function exportReport(formName, actionURL) {
  theForm = document.forms[formName];
  if (theForm) {
    if (theForm._xf.value == 'analyze') {
      window.alert(ANALYZER_TEMPLATE_OPERATION_NOT_SUPPORTED);
      return false;
    }
    ...
  }
}
```

Rồi, đưa về cho sysadmin để test thử, bên kia báo lại mỗi IE trên Windows XP
thì không được, còn lại thì OK. Search thử thì thấy IE 8 không support cái hàm
`getElementsByClassName`, chỉnh lại một xíu ở đây, thay vì tìm các element bởi
`class` thì check tất cả `tagName` trong form `xdoRptForm`, cái nào có
`tagName==BUTTON` thì disable nó đi, code như sau

```javascript
var form = document.getElementById("xdoRptForm");
for (var i=0; i < form.length; i++) {
    if (form[i].tagName == "BUTTON") {
        form[i].disabled = true;
        form[i].style.background='#d8d8d8';
    }
}
```

Xong rồi, giờ thì cả làng vui vẻ, các em gái xinh đẹp ở ngoài Khối CNNH không
còn làm phiền sysadmin nữa.

## #1 Script AutoDel SAD

Ngân hàng đang muốn lấy chứng chỉ PCI DSS, có 1 task liên quan đến việc xóa các
dữ liệu nhạy cảm của khách hàng, trên cả phía client và máy chủ file-server,
tất nhiên là sau khi bên nghiệp vụ đã xử lý xong và không cần dùng nữa. Nếu để
bên nghiệp vụ thực hiện xóa bằng tay thì không đảm bảo có bỏ sót xóa ngày nào
hay không? Giải pháp ở đây là làm script tự động thực hiện việc này sau xx giờ.

Nếu xx nhỏ, nhỏ như thế nào thì không rõ, và nếu, lại nếu, bên nghiệp vụ chưa
hoàn tất việc của họ (cụ thể ở đây là dập thẻ), mà script đã xóa mất dữ liệu,
lúc này thì mệt đây, phải tạo lại dữ liệu mới cho khách hàng, mà việc này
không hề đơn giản. Giải pháp là tăng xx lên, lên bao nhiêu cũng chưa rõ. Ngoài
ra, script có khả năng hoãn việc xóa file.

Tóm lại, yêu cầu kỹ thuật là:

* Script tự động xóa các tập tin được tạo ra từ xx giờ trước
* Có thể hoãn thực hiện việc xóa file
* Xóa -một cách an toàn- tập tin, theo tiêu chuẩn DoD 5220.22-M
* Chạy trên HDH của client và file-server

File-server ở đây là Solaris, còn client là Windows. Trên Solaris, việc lấy
thời gian tạo ra file lằng nhằng hơn so với RHEL, ở đây mình dùng lệnh `truss`

```bash
_tmp=$(mktemp)
_mt=$(truss -vlstat -tlstat -o $_tmp ls -l $1)
_crtime=$(cat $_tmp | grep 'mt = ' | cut -d '[' -f2 | awk '{print $1}')
```

Đối với yêu cầu xóa file theo chuẩn DoD 5220.22-M 3-pass, đọc xem chuẩn này như
thế nào?

```
The DoD 5220.22-M data sanitization method is usually implemented in the
following way:

* Pass 1: Writes a zero and verifies the write
* Pass 2: Writes a one and verifies the write
* Pass 3: Writes a random character and verifies the write
```

Mình chơi hẳn 7-pass luôn, code như sau (lưu ý là blocksize bằng filesize)

```bash
 _filesize=$(ls -l $1 | awk '{print $5}')

# 3-PASS
dd if=/dev/zero bs=$_filesize count=1 of=$1
dd if=/dev/zero bs=$_filesize count=1 | tr '\0' '\61' > $1
dd if=/dev/random bs=$_filesize count=1 of=$1

# 7-PASS
dd if=/dev/zero bs=$_filesize count=1 of=$1
dd if=/dev/zero bs=$_filesize count=1 | tr '\0' '\62' > $1
dd if=/dev/random bs=$_filesize count=1 of=$1
dd if=/dev/zero bs=$_filesize count=1 of=$1

rm $1
```

Lưu ý thêm, cái này chỉ là ghi đè dữ liệu lên, nên phải có bước cuối cùng là
xóa cái file đó đi.

Trên Windows thì dễ rồi, cài thêm cái Eraser vào để nó xóa, script của mình chỉ
việc check xem cần xóa file nào, rồi quăng cho nó xóa, hết phim.
