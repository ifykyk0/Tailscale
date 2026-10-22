# JioRouter AX6000 + OpenWrt par Tailscale setup (step by step)

Is guide se aap **ghar se bahar (mobile data ya kisi bhi WiFi par)** se ye dono panel khol paoge:

- **Apna OpenWrt router** (`192.168.1.1`)
- **Jio ODU / AirFiber unit** jisse router juda hai (aam taur par `192.168.225.1`)

Isme kisi ko port-forwarding ya public IP ki zaroorat nahi hai. Sab kuch **Tailscale** (free personal plan) se hota hai.

```
Aapka phone (Tailscale ON, mobile data)
        |
        |  Tailscale (encrypted tunnel)
        v
   OpenWrt router  ---->  192.168.1.1    (router ka panel)
        |
        +-------------->  192.168.225.1  (Jio ODU ka panel)
                               |
                            Internet
```

Router ek "rasta" ban jata hai: phone Tailscale se router tak pahunchta hai, aur router aage Jio ODU tak pahucha deta hai.

**Time lagega:** lagbhag 20-30 minute. Har step ke baad check karte chalo, jaldi mat karo.

> Ye ek unofficial community guide hai. Jio, Tailscale ya OpenWrt se iska koi sambandh nahi hai. Apne risk par use karo, aur pehle config ka backup lo.

**Kahan test hua hai:** JioRouter AX6000 (JIDU6401 / 6J01), OpenWrt 25.12.6, Tailscale 1.104, `luci-app-tailscale-community`.

---

## Shuru karne se pehle (5 minute)

1. **Router ka root password set karo.** SSH ya LuCI me **System → Administration** me jaake password daalo. Bina password ke router ka panel LAN par kisi ko bhi khula hai.
2. **Config backup lo.** LuCI me **System → Backup / Flash Firmware → Generate archive**. Jo file download ho use PC me safe rakho.
3. **Tailscale account banao** (free): `https://login.tailscale.com` par Google, Microsoft ya GitHub se sign up karo.
4. Apne **phone me Tailscale app** install karo (App Store / Play Store) aur usi account se login karo.
5. Router ko **LAN cable** se laptop se jodo, aur `http://192.168.1.1` kholkar login karo.

---

## Step 1: Tailscale packages install karo

Agar aapke firmware me `tailscale` aur `luci-app-tailscale-community` pehle se hain to **Step 2 par jao**. Check karne ke liye LuCI me **Services** menu dekho, usme **Tailscale** hona chahiye.

Nahi hai to router par SSH karo (Windows me PowerShell kholkar `ssh root@192.168.1.1`) aur ye chalao:

```
apk update
apk add tailscale luci-app-tailscale-community
```

Phir LuCI page refresh karo (Ctrl + F5). **Services → Tailscale** dikhna chahiye.

> Dhyan rakho: `luci-app-tailscale` (community ke bina wala) purana package hai jo ab 25.12 ke feed me nahi hai. Ussi ko `luci-app-tailscale-community` se badalna hai.

---

## Step 2: Router ko Tailscale me login karo

1. LuCI me **Services → Tailscale** kholo.
2. **Account Settings** tab me **Login** button dabao.
3. Ek link aayega. Use naye browser tab me kholo, usi Tailscale account se sign in karo, aur **Connect / Approve** dabao.
4. Router wale page ko **Ctrl + F5** se refresh karo. Status **LOGGED OUT** se badalkar running/connected dikhna chahiye, aur ek `100.x.x.x` IP bhi dikhega.

Login link ya auth key **kisi ko share mat karna**.

---

## Step 3: Jo raaste kholne hain wo bata do

LuCI me **Services → Tailscale → General Settings** par jao.

| Setting | Kya rakhna hai |
|---|---|
| Firewall Mode | `nftables` (jaisa hai waisa) |
| Accept Routes | band |
| Advertise Exit Node | band (zaroorat ho to neeche "Optional: Exit Node" dekho) |
| Allow LAN Access | band |
| Enable Web Interface | band |
| Disable SNAT | **band rakho** (ye zaroori hai) |
| Shields Up | **band rakho** (on kiya to bahar se kuch nahi khulega) |
| Enable Tailscale SSH | band |
| Disable MagicDNS | band |
| Exit Node | None |

**Advertise Routes** me ye do cheezein daalo (dropdown se chuno ya type karke Enter dabao):

- `192.168.1.0/24` (aapke router ka ghar wala network)
- `192.168.225.1/32` (sirf Jio ODU ka ek address)

> **ODU ka IP kaise pata kare?** LuCI me **Status → Overview** me WAN ka **Gateway** dekho, ya router par SSH me `ip route` chalao. `default via ...` ke saamne jo IP hai wahi ODU ka hai (aam taur par `192.168.225.1`). Alag ho to wahi `/32` ke saath daalo. ODU ke peeche ka poora `/24` advertise mat karo, sirf ek IP kaafi hai.

Neeche **Save & Apply** dabao.

---

## Step 4: Firewall theek karo

1. Usi Tailscale page par **Auto Configure Firewall** button dabao.
2. LuCI me **Network → Firewall** kholo, `tailscale` zone ke saamne **Edit** dabao:
   - **Input:** `accept`
   - **Allow forward to destination zones:** `lan` **aur `wan`** dono chuno
   - **Allow forward from source zones:** sirf `lan` (isme `wan` mat jodna)
   - **IPv4 Masquerading** aur **MSS clamping:** on
3. **Save**, phir page ke neeche **Save & Apply**.

`wan` zone me **Masquerading** pehle se on hota hai, use band mat karna.

**Alternative (command se):** agar upar wala forward `wan` wala option na dikhe, to SSH me ye chalao:

```
uci set firewall.tailscale_to_wan='forwarding'
uci set firewall.tailscale_to_wan.src='tailscale'
uci set firewall.tailscale_to_wan.dest='wan'
uci commit firewall
/etc/init.d/firewall restart
```

---

## Step 5: Tailscale website me routes approve karo (ye zaroori hai)

Isi step ko sabse zyada log bhool jaate hain.

1. `https://login.tailscale.com` kholo → **Machines**.
2. Apne router (naam aam taur par `openwrt`) ke saamne **...** (teen dots) dabao.
3. **Edit route settings...** → dono routes (`192.168.1.0/24` aur `192.168.225.1/32`) **tick** karo → **Save**.
4. Wahi **...** menu me **Disable key expiry** dabao. Nahi to kuch mahine baad router apne aap logout ho jayega.

Router ke naam ke neeche **Subnets** aur **Expiry disabled** ke badge dikhne chahiye.

---

## Step 6: Test karo

1. Phone me **WiFi band** karo aur **mobile data on** karo.
2. Tailscale app me VPN **on** karo.
3. Safari / Chrome me kholo:
   - `http://192.168.1.1` → router ka LuCI panel
   - `https://192.168.225.1` → Jio ODU ka login page

Jio ODU ka page **HTTPS** par hai aur uska certificate self-signed hai, isliye browser "Not secure" ya "not private" ka warning dega. Ye normal hai, **Advanced / Show Details → Proceed / visit this website** dabao.

Router ka panel `http://<router-ka-100.x.x.x-IP>` se bhi hamesha khul sakta hai (IP Tailscale app ya website me dikhta hai).

---

## Optional: Exit Node (poora internet ghar ke Jio se chalana)

Isse aapke phone ka **saara internet** ghar ke connection se guzarta hai (jaise public WiFi par safe rehne ke liye). **Sirf panel kholne ke liye iski zaroorat nahi hai**, aur ye speed kam kar sakta hai.

1. LuCI → **Services → Tailscale** me **Advertise Exit Node** tick karo, **Save & Apply**.
2. Tailscale website → Machines → router → **...** → **Edit route settings** → **Use as exit node** tick karo → **Save**.
3. Phone ke Tailscale app me **Exit Node** me router chuno.

Jab zaroorat na ho to app me Exit Node **None** kar do. Warna agar router ya Jio band hua to phone ka internet bhi band ho jayega.

---

## Dikkat aaye to (troubleshooting)

| Problem | Kya karo |
|---|---|
| Router me **Services → Tailscale** nahi dikhta | Step 1 karo, phir Ctrl + F5 |
| Status **LOGGED OUT** | Step 2 dobara karo. Firmware flash ke baad aksar dobara login karna padta hai |
| `192.168.1.1` bahar se nahi khulta | Step 5 me route approve hua? Phone ke Tailscale app me subnets/routes allowed hain? |
| Router khulta hai par **ODU nahi** | Step 4 ka `tailscale → wan` forward check karo, aur URL **https** se kholo |
| ODU ka IP alag hai | `ip route` se gateway dekho, wahi `/32` ke saath daalo |
| Jis jagah ho wahan ka local network bhi `192.168.1.x` ya `192.168.225.x` hai | Wahan ka local device khulega, ghar wala nahi. Mobile data par aisa nahi hota |
| `apk update` me **Operation not permitted** | Router ka internet abhi stable nahi hota, 1-2 minute ruk ke dobara chalao. `ping 8.8.8.8` chala ke dekho |
| **Safari par ODU atak jata hai** | Laptop ke Chrome se bhi try karo, aur Step 4 firewall check karo |

Router par sab kuch ek saath dekhne ke liye:

```
tailscale status
tailscale debug prefs | grep -A2 AdvertiseRoutes
ip route
```

---

## Reboot test (zaroor karo)

Sab kaam kar jaye to router ko ek baar `reboot` karo aur 2 minute baad phir phone se `192.168.1.1` aur `https://192.168.225.1` kholkar dekho. Isse pakka hota hai ki login, routes aur firewall reboot ke baad bhi bane rehte hain.

---

## Security ke liye zaroori baatein

- **Router ka password** strong rakho. **Jio ODU ka default password badal do**, kyunki ab ye Tailscale se bhi pahunch me hai.
- Apna **Tailscale network kisi ko share mat karo**, aur **Tailscale auth key / login link** kisi ko mat bhejo.
- **Shields Up** band rakha hai, lekin ye zaroori hai ki aapke Tailscale network me sirf aapke apne devices hon. Tailscale website → **Access controls** se ye aur limit ki ja sakti hai.
- **LuCI (router ka panel) ko internet par kabhi expose mat karo.** Bahar se sirf Tailscale ke raste hi kholo.
- Router ka **config backup** (`.tar.gz`) aur **MFG / Reserved / Factory** partition ka backup (agar liya ho) private rakho. Inme WiFi password aur device ka private data hota hai, isliye inhe GitHub ya kisi public jagah par kabhi upload mat karo.
- Apni guide ya screenshots GitHub par daalne se pehle unme se **apna Tailscale IP, MAC address, serial number, WiFi password aur login link** hata do.

---

## Wapas hatana ho to

- Router par: `tailscale down`
- Tailscale website → Machines → router → **...** → **Delete**
- LuCI → **Network → Firewall** me `tailscale` zone aur forwarding rule hata do

---

## Credits

OpenWrt, Tailscale aur `luci-app-tailscale-community` ke developers ko dhanyavaad. Ye guide ek real JioRouter AX6000 par kiye gaye setup se bani hai.
