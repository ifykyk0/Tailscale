# JioRouter AX6000 (OpenWrt) par Tailscale: bahar se Router aur Jio ODU dono kholo

Is guide se aap **ghar se bahar (mobile data ya kisi bhi WiFi par)** se apne ghar ke ye dono panel khol paoge:

- **OpenWrt router** ka panel: `http://192.168.1.1`
- **Jio ODU (AirFiber unit)** ka panel: `https://192.168.225.1` (aam taur par; apna IP Step 5 me pakka karenge)

Koi port-forwarding nahi, koi public IP nahi, aur router internet par khula nahi chhodna padta. Sab kuch **Tailscale** (free personal plan) se hota hai.

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

> Ye ek unofficial community guide hai. Jio, Tailscale ya OpenWrt se iska koi sambandh nahi hai. Apne risk par use karo.
>
> **Kahan test hua hai:** JioRouter AX6000 (JIDU6J01 / JIDU6401), OpenWrt 25.12.6, Tailscale 1.104, `luci-app-tailscale-community`, iPhone. Doosre version me menu ke naam thode alag ho sakte hain.

**Time lagega:** lagbhag 30-40 minute. Har step ke baad "Kya dikhna chahiye" wali line dekho, aur tabhi aage badho jab wo dikh jaye.

---

## Zaroori shabdon ka matlab (ek baar padh lo)

| Shabd | Matlab |
|---|---|
| **Router** | Aapka OpenWrt wala JioRouter AX6000 |
| **Jio ODU** | Jio AirFiber ki outdoor unit, jisse aapka router internet leta hai |
| **LuCI** | Router ka web panel (`http://192.168.1.1`) |
| **SSH** | Router me commands type karne ka tareeka (Windows me PowerShell se) |
| **Tailscale** | Free app jo aapke phone aur router ke beech ek surakshit (encrypted) raasta banata hai |
| **Admin page** | Tailscale ki website: `https://login.tailscale.com/admin/machines` |
| **Subnet route** | Router ko bolna: "mujhe is address tak pahuchne ka rasta bana do" |
| **Exit node** | Phone ka poora internet router ke raaste chalana (optional, Step 11) |

---

## Kya chahiye (checklist)

- [ ] Router par **OpenWrt 25.12** chal raha ho
- [ ] Router **internet** se juda ho (Jio ODU se)
- [ ] Ek **laptop/PC** jo router ke WiFi ya LAN cable se juda ho
- [ ] Ek **Tailscale account** (free, Google/Microsoft/GitHub se ban jata hai)
- [ ] **Phone** (iPhone ya Android) jisme Tailscale app install kar sako
- [ ] 30-40 minute ka shaanti ka time

---

## SSH kaise kholte hain (jab bhi guide me "SSH karo" likha ho)

**Windows:**
1. Keyboard par **Windows key** dabao, `PowerShell` type karo, **Enter** dabao.
2. Ye likho aur Enter dabao:
   ```
   ssh root@192.168.1.1
   ```
3. Pehli baar "Are you sure you want to continue connecting" poochega. `yes` type karke Enter dabao.
4. **Password** poochega. Wo router ka password hai (Step 0 me set karoge). Type karte waqt **kuch dikhta nahi**, ye normal hai. Type karke Enter dabao.
5. `root@OpenWrt:~#` jaisa kuch dikhe to aap router ke andar ho.

**Mac / Linux:** Terminal kholo aur wahi `ssh root@192.168.1.1` chalao.

**Commands kaise chalayein:** guide me jo code ke dabbe me likha hai, use copy karo aur SSH wale window me **right-click** karke paste karo (Windows PowerShell me right-click paste karta hai), phir Enter dabao.

> Dhyan rakho: SSH wale window me jo `root@OpenWrt:~#` dikhta hai, wahi router ka terminal hai. Windows ke commands (jaise `scp`) alag window me chalte hain. Is guide me hum sirf router ke commands use karte hain.

---

## Step 0: Pehle ye 2 kaam karo (router ko surakshit karo)

### 0A. Router ka password set karo
Router par agar password nahi hai to LAN ya WiFi se juda koi bhi router ko control kar sakta hai.

1. Laptop me browser kholo aur `http://192.168.1.1` likho.
2. Upar menu me **System → Administration** par jao.
3. **Router Password** me naya password do (lamba aur yaad rakhne layak, jaise 3-4 random words). Dono boxes me wahi likho.
4. Neeche **Save** dabao.

**Kya dikhna chahiye:** upar jo peela "No password set!" wala warning tha, wo hat jayega.

### 0B. Config ka backup lo
1. **System → Backup / Flash Firmware** kholo.
2. **Generate archive** dabao.
3. Jo `backup-....tar.gz` file download ho, use apne PC me kisi safe folder me rakh lo.

> Ye file kisi ko mat bhejna aur GitHub par upload mat karna. Isme aapke WiFi password hote hain.

---

## Step 1: Tailscale account aur phone app

1. Browser me `https://login.tailscale.com` kholo. **Sign up** karo (Google, Microsoft ya GitHub se).
2. Phone me **Tailscale** app install karo (App Store ya Play Store).
3. Phone me usi account se **login** karo, aur jab VPN permission maange to **Allow** karo.

**Kya dikhna chahiye:** `https://login.tailscale.com/admin/machines` par abhi aapka **phone** ek machine ke roop me dikhega (green dot ke saath).

---

## Step 2: Router me Tailscale hai ya nahi

1. Router ka panel kholo (`http://192.168.1.1`).
2. Upar menu me **Services** par click karo.
3. List me **Tailscale** dikhta hai?

- **Haan dikhta hai:** Step 3 par jao.
- **Nahi dikhta:** neeche wale commands se install karo.

**Install karna (SSH karke):**

```
apk update
apk add tailscale luci-app-tailscale-community
```

Install ho jaye to browser me **Ctrl + F5** dabao (page refresh) aur **Services** menu me dobara dekho.

> Dhyan rakho: purana package `luci-app-tailscale` ab OpenWrt 25.12 ke official feed me nahi hai. Uski jagah `luci-app-tailscale-community` lagana hai.
>
> Agar `apk update` me **"Operation not permitted"** aaye, to 1-2 minute ruko (router ka internet abhi connect ho raha hota hai) aur dobara chalao. Phir bhi na chale to `ping -c 3 8.8.8.8` chalao. Reply na aaye to router ka internet hi band hai.
>
> Agar aap apna custom firmware bana rahe ho (Firmware Selector se), to package list me `tailscale` aur `luci-app-tailscale-community` daal do.

**Kya dikhna chahiye:** **Services → Tailscale** page khulta hai, aur upar **Service Status: LOGGED OUT** likha hota hai.

---

## Step 3: Router ko Tailscale me login karo

1. **Services → Tailscale** kholo.
2. **Account Settings** tab par click karo (General Settings ke bagal me).
3. **Login** button dabao.
4. Ek **lamba link** dikhega. Usse copy karke **naye browser tab** me kholo.
5. Tailscale ka page khulega. **Usi account** se sign in karo jisse phone me kiya tha, aur **Connect** ya **Approve** dabao.
6. Router wale page par wapas aao aur **Ctrl + F5** dabao.

**Kya dikhna chahiye:**
- Router ke page par Status ab **LOGGED OUT** nahi dikhta. Running ya connected jaisa dikhta hai, aur ek `100.x.x.x` wala IP bhi dikhta hai.
- Admin page (`https://login.tailscale.com/admin/machines`) par ab **2 machines** dikhte hain: aapka phone aur `openwrt`, dono ke aage **green dot**.

> Login link sirf aapke liye hai. Ise kisi ko mat bhejo.
>
> Agar admin page par purani `openwrt` entry bhi dikhe jiska dot grey hai (pehle kisi firmware ki), to uske saamne **...** → **Delete** kar do. Phone wali entry mat hatana.

---

## Step 4: Key expiry band karo (nahi to router kuch mahine baad logout ho jayega)

1. `https://login.tailscale.com/admin/machines` kholo.
2. `openwrt` ke saamne wale **...** (teen dots) par click karo.
3. **Disable key expiry** dabao.

**Kya dikhna chahiye:** machine ke naam ke neeche **Expiry disabled** ka badge dikhta hai.

---

## Step 5: Jio ODU ka IP pata karo

Zyadatar Jio ODU ka IP `192.168.225.1` hota hai, par apna pakka kar lo, kyunki ye alag ho sakta hai.

**Tareeka 1 (panel se):** LuCI me **Status → Overview** kholo. Neeche **Network** section me WAN ka **Gateway** dekho. Wahi ODU ka IP hai.

**Tareeka 2 (SSH se):** ye command chalao:
```
ip route
```
Jo line `default via ...` se shuru hoti hai, uske saamne wala IP ODU ka hai. Jaise `default via 192.168.225.1 dev wan` me ODU ka IP `192.168.225.1` hai.

**Check karo:** laptop ko router ke WiFi se jodo aur browser me `https://<wo-IP>` kholo (jaise `https://192.168.225.1`). "Not secure" ka warning aayega, **Advanced → Proceed** dabao. **JioODU** wala login page (Jio ka logo aur username/password) dikhna chahiye. Dikhe to IP sahi hai.

Isi IP ko neeche `<ODU-IP>` likha hai.

---

## Step 6: Router ki Tailscale settings

1. **Services → Tailscale → General Settings** tab kholo.
2. Ye settings rakho (zyadatar pehle se sahi hoti hain, bas check karo):

| Setting | Kya rakhna hai | Kyun |
|---|---|---|
| Firewall Mode | **nftables** | Aapke router ka firewall nftables hai |
| Accept Routes | band | Iski zaroorat nahi |
| Advertise Exit Node | band | Sirf optional Step 11 me chahiye |
| Allow LAN Access | band | Exit node ke liye hai |
| Enable Web Interface | band | Extra panel khulta hai, zaroorat nahi |
| Disable SNAT | **band** (tick mat karo) | Isse JioODU ko jawab sahi raaste se milta hai |
| Shields Up | **band** (tick mat karo) | Tick kiya to bahar se kuch nahi khulega |
| Enable Tailscale SSH | band | Normal SSH hi chalega |
| Disable MagicDNS | band | Jaisa hai rehne do |
| Exit Node | **None** | Abhi nahi chahiye |

3. Neeche **Advertise Routes** me **do cheezein** daalo:
   - Dropdown me se **`192.168.1.0/24`** chuno (ye router ka ghar wala network hai).
   - Dusre box me type karo **`<ODU-IP>/32`**, jaise `192.168.225.1/32`, aur **Enter** dabao. Wo ek chip (chhota tag) ban jayega.

> **`/32` ka matlab sirf ek address.** Dropdown me `192.168.225.0/24` bhi dikhega, use **mat chunna**. Usse ODU ke peeche ke sab devices khul jate hain, jabki hume sirf ODU ka panel chahiye.

4. Page ke neeche **Save & Apply** dabao.

**Kya dikhna chahiye:** Advertise Routes me do chip dikhte hain: `192.168.1.0/24` aur `192.168.225.1/32` (ya aapka ODU IP).

---

## Step 7: Firewall theek karo

1. Usi **Services → Tailscale** page par **Auto Configure Firewall** button dabao. Ye `tailscale` naam ka network aur firewall zone bana deta hai.
2. **Network → Firewall** kholo.
3. `tailscale` zone ke saamne **Edit** dabao.
4. Ye settings rakho:

| Setting | Value |
|---|---|
| Input | **accept** |
| Output | **accept** |
| Intra zone forward | **accept** |
| IPv4 Masquerading | tick |
| MSS clamping | tick |
| Covered networks | **tailscale** |
| Allow forward to destination zones | **lan** aur **wan** dono |
| Allow forward from source zones | **sirf lan** (wan mat jodo) |

5. Neeche **Save** dabao. Phir firewall page ke neeche **Save & Apply**.

> **Kyun `wan` chahiye?** Jio ODU router ke **WAN** side par hai. `wan` ko destination me jodne se Tailscale se aaya traffic ODU tak ja sakta hai. "Source zones" me `wan` nahi jodna, nahi to ODU ki taraf se aapke Tailscale network me connection khul jayega.
>
> `wan` zone ki **Masquerading** pehle se on hoti hai. Use band mat karna.

**Command se karna ho (optional):** agar upar ki settings samajh na aaye, ya `wan` forward na dikhe, to SSH me ye chalao:

```
uci set firewall.tailscale_to_wan='forwarding'
uci set firewall.tailscale_to_wan.src='tailscale'
uci set firewall.tailscale_to_wan.dest='wan'
uci commit firewall
/etc/init.d/firewall restart
```

**Kya dikhna chahiye:** firewall restart ke baad koi error nahi aata.

---

## Step 8: Admin page me routes approve karo (ye step log sabse zyada bhoolte hain)

Router ne routes maange hain, par Tailscale tabhi use karta hai jab aap admin page par haan bolo.

1. `https://login.tailscale.com/admin/machines` kholo.
2. `openwrt` ke saamne **...** par click karo.
3. **Edit route settings...** dabao.
4. **Subnet routes** me dono routes **tick** karo:
   - `192.168.1.0/24`
   - `<ODU-IP>/32` (jaise `192.168.225.1/32`)
5. **Save** dabao.

**Kya dikhna chahiye:** `openwrt` ke naam ke neeche **Subnets** ka badge (neela) aur **Expiry disabled** ka badge dikhte hain. Subnets na dikhe to page refresh karo.

---

## Step 9: Phone se test karo

1. Phone me **WiFi band** karo aur **mobile data on** karo. (Isse pakka hota hai ki aap sach me bahar se test kar rahe ho.)
2. **Tailscale app** kholo aur VPN **on** karo. App me router (`openwrt`) "Connected" dikhna chahiye.
3. Tailscale app ki settings me **Use Tailscale subnets** on hona chahiye (agar option dikhe).
4. Phone ke browser (Safari/Chrome) me kholo:

| Kya kholna hai | Address |
|---|---|
| Router ka panel | `http://192.168.1.1` |
| Jio ODU ka panel | `https://<ODU-IP>` (jaise `https://192.168.225.1`) |

**Kya dikhna chahiye:**
- `http://192.168.1.1` par **OpenWrt ka login page**.
- `https://192.168.225.1` par **JioODU ka login page** (Jio ka orange logo, username aur password).

> **"Not secure" ka warning:** Jio ODU ka certificate self-signed hai, isliye browser ek warning dega. Ye is apne device ke liye normal hai. Safari me **Show Details → visit this website**, Chrome me **Advanced → Proceed**. Ye sirf ODU ke liye karo.
>
> **Router ka doosra address:** router ka panel `http://<router-ka-100.x.x.x-IP>` se bhi khulta hai. Ye IP Tailscale app ya admin page me router ke saamne dikhta hai. Ye address route ke bina bhi kaam karta hai, to pehle ise try karo agar `192.168.1.1` na khule.

---

## Step 10: Reboot test aur final backup

1. Router ko reboot karo: **System → Reboot → Perform reboot**.
2. 2-3 minute ruko.
3. Phone se Step 9 dobara karo.

Dono panel khulen to setup pakka hai.

Phir **System → Backup / Flash Firmware → Generate archive** se working config ka naya backup le lo.

---

## Step 11 (optional): Exit Node

Isse phone ka **poora internet** ghar ke Jio connection se guzarta hai (jaise public WiFi par). **Sirf panel kholne ke liye iski zaroorat nahi hai.** Exit node on rehne par speed kam ho sakti hai, aur router ya ghar ka internet band hua to phone ka internet bhi band ho jayega.

1. **Services → Tailscale → General Settings** me **Advertise Exit Node** tick karo, **Save & Apply** dabao.
2. Admin page par `openwrt` ke **...** → **Edit route settings...** → **Use as exit node** tick karo → **Save**.
3. Phone ke Tailscale app me **Exit Node** kholo aur `openwrt` chuno. **Allow Local Network Access** tick rakho.

Band karne ke liye phone me Exit Node **None** chuno.

---

## Dikkat aaye to (troubleshooting)

| Dikkat | Kya karo |
|---|---|
| Router me **Services → Tailscale** nahi dikhta | Step 2 karo, phir Ctrl + F5 |
| Status **LOGGED OUT** | Step 3 dobara karo. Naya firmware flash karne ke baad aksar dobara login karna padta hai |
| Admin page me **Subnets** badge nahi | Step 8 karo, routes approve nahi hue |
| `192.168.1.1` bahar se nahi khulta | Phone me Tailscale ON hai? Mobile data par ho? Step 6 aur 8 check karo. Pehle `http://<router-ka-100.x.x.x-IP>` try karo |
| `192.168.1.1` khulta hai par **JioODU nahi** | Step 7 ke command wale forward rule ko chalao. Browser me `https://` use karo (`http` nahi). Laptop ke Chrome se bhi try karo |
| Jis network par ho wahan bhi `192.168.1.x` ya `192.168.225.x` hai | Browser wahan ke local device ko kholega, ghar wale ko nahi. Mobile data par ye dikkat nahi aati. Is case me `http://<router-ka-100.x.x.x-IP>` use karo |
| JioODU ka IP badal gaya | Step 5 se naya IP nikalo aur Step 6 aur 8 me naya `/32` daalo |
| `apk update` me **Operation not permitted** | 1-2 minute ruk ke dobara chalao. `ping -c 3 8.8.8.8` chalake internet check karo |

**Router par ye commands chala ke dekho (SSH me):**

```
tailscale status
tailscale debug prefs | grep -A2 AdvertiseRoutes
ip route
```

- `tailscale status` me router aur phone dono dikhne chahiye.
- Dusre command me aapke dono routes dikhne chahiye.
- `ip route` me `default via ... dev wan` (ODU ka IP) dikhna chahiye.

---

## Surakshit rehne ke liye

- Router ka **password strong** rakho. **Jio ODU ka default password badal do**, kyunki ab ye Tailscale se bhi pahunch me hai.
- Apna **Tailscale network (tailnet) kisi ko share mat karo**, aur router ka login link ya auth key kisi ko mat bhejo.
- Tailscale ke network me **sirf apne devices** rakho. Zyada control chahiye to admin page ke **Access controls** me limits laga sakte ho.
- **Router ka panel (LuCI) internet par kabhi expose mat karo** (port forwarding ya DMZ se). Bahar se sirf Tailscale ke raaste kholo.
- Sirf **`/32`** route advertise karo ODU ke liye. Poora subnet mat do.
- Router ka **config backup** (`.tar.gz`) aur agar partition backup liya ho to wo bhi **private** rakho. Inhe GitHub ya kisi public jagah par kabhi upload mat karo.
- Guide ya screenshots GitHub par daalne se pehle unme se apna **Tailscale IP, MAC address, serial number, WiFi password aur login link** hata do.

---

## Wapas hatana ho to

1. Router par: `tailscale down`
2. Admin page par `openwrt` ke **...** → **Delete**
3. LuCI → **Network → Firewall** me `tailscale` zone aur forwarding rule hata do
4. Advertise Routes ke dono chip hata do aur **Save & Apply** karo

---

## Aksar poochhe jane wale sawal

**Kya ye free hai?** Haan, Tailscale ka personal plan free hai.

**Kya Jio AirFiber par chalega, jahan public IP nahi hota?** Haan. Tailscale ko port-forwarding ya public IP nahi chahiye, kyunki connection aapke devices apne aap bana lete hain.

**Phone ki battery par asar?** Thoda hota hai. Zaroorat na ho to Tailscale app me VPN off kar do.

**Exit node zaroori hai?** Nahi. Sirf router aur ODU ke panel kholne ke liye Step 1-10 kaafi hain.

---

## Credits

OpenWrt, Tailscale aur `luci-app-tailscale-community` ke developers ko dhanyavaad. Ye guide ek real JioRouter AX6000 par kiye gaye setup se bani hai.
