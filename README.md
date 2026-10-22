# JioRouter AX6000 (OpenWrt) par Tailscale: bahar se Router aur Jio ODU dono kholo

*(English summary neeche hai / English summary below)*

Is guide se aap **ghar se bahar (mobile data ya kisi bhi WiFi par)** se apne ghar ke ye dono panel khol paoge:

- **OpenWrt router** ka panel: `http://192.168.1.1`
- **Jio ODU (AirFiber unit)** ka panel: `https://192.168.225.1` (aam taur par; apna IP Step 5 me pakka karenge)

Koi port-forwarding nahi, koi public IP nahi, aur router internet par khula nahi chhodna padta. Sab kuch **Tailscale** (free personal plan) se hota hai.

> **Last tested:** October 2026. JioRouter AX6000 (JIDU6J01 / JIDU6401), OpenWrt 25.12.6, Tailscale 1.104, `luci-app-tailscale-community`, **iPhone par**.
>
> **Android par abhi test nahi hua.** Wahan bhi aise hi chalna chahiye, par app ke menu ke naam alag ho sakte hain.
>
> Ye ek unofficial community guide hai. Jio, Tailscale ya OpenWrt se iska koi sambandh nahi hai. Apne risk par use karo.

---

## TL;DR: 10 line me poora kaam

1. Router ka **password set** karo, aur **config backup** lo (Step 0).
2. Tailscale ka **free account** banao aur phone me **app** lagao (Step 1).
3. Router ke menu me **Services → Tailscale** dikhe, nahi to install karo (Step 2).
4. Router ko Tailscale me **login** karo (Step 3), aur **key expiry band** karo (Step 4).
5. **Jio ODU ka IP** pata karo, aam taur par `192.168.225.1` (Step 5).
6. Router par **Advertise Routes** me do cheezein daalo: `192.168.1.0/24` aur `<ODU-IP>/32` (Step 6).
7. **Firewall** me `tailscale` zone se `wan` tak forward allow karo (Step 7).
8. Tailscale ki website par dono routes **approve** karo (Step 8). **Ye step sabse zyada log bhoolte hain.**
9. Phone par WiFi band, mobile data on, Tailscale on, phir dono address kholo (Step 9).
10. Router **reboot** karke ek baar phir test karo (Step 10).

Pura detail neeche hai. Atko to Step ke "Kya dikhna chahiye" wali line dekho.

---

## Ye kaam kaise karta hai

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

**Time lagega:** lagbhag 30-40 minute. Har step ke baad "Kya dikhna chahiye" wali line dekho, aur tabhi aage badho jab wo dikh jaye.

---

## Zaroori shabdon ka matlab (ek baar padh lo)

| Shabd | Matlab |
|---|---|
| **Router** | Aapka OpenWrt wala JioRouter AX6000 |
| **Jio ODU** | Jio AirFiber ki outdoor unit, jisse aapka router internet leta hai |
| **LuCI** | Router ka web panel (`http://192.168.1.1`) |
| **SSH** | Router me commands type karne ka tareeka (Windows me PowerShell se) |
| **Tailscale** | Free app jo aapke devices ke beech ek surakshit (encrypted) raasta banata hai |
| **Tailnet** | Aapke Tailscale account ke saare devices ka apna private network |
| **Admin page** | Tailscale ki website: `https://login.tailscale.com/admin/machines` |
| **Subnet route** | Router ko bolna: "mujhe is address tak pahuchne ka rasta bana do" |
| **`/32`** | Sirf **ek** address. Jaise `192.168.225.1/32` matlab sirf ODU |
| **`100.x.x.x` IP** | Tailscale ne aapke device ko jo apna address diya, jaise `100.101.102.103` |
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

Zyadatar steps browser me hote hain. SSH sirf Step 2 (agar Tailscale install karna ho) aur dikkat aane par chahiye.

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

> **Agar "REMOTE HOST IDENTIFICATION HAS CHANGED" ka error aaye:** ye tab hota hai jab router ka firmware naya flash hua ho. Apne PC ke PowerShell me ye chalao, phir dobara SSH karo (`yes` likho):
> ```
> ssh-keygen -R 192.168.1.1
> ```

---

## Step 0: Pehle ye 2 kaam karo (router ko surakshit karo)

### 0A. Router ka password set karo
Router par agar password nahi hai to LAN ya WiFi se juda koi bhi router ko control kar sakta hai.

1. Laptop me browser kholo aur `http://192.168.1.1` likho.
2. Upar menu me **System → Administration** par jao.
3. **Router Password** me naya password do (lamba aur yaad rakhne layak, jaise 3-4 random words). Dono boxes me wahi likho.
4. Neeche **Save** dabao.

**Kya dikhna chahiye:** upar jo peela "No password set!" wala warning tha, wo hat jayega.

<!-- ![Step 0A: Router Password](screenshots/step-00a-router-password.png) -->

### 0B. Config ka backup lo
1. **System → Backup / Flash Firmware** kholo.
2. **Generate archive** dabao.
3. Jo `backup-....tar.gz` file download ho, use apne PC me kisi safe folder me rakh lo.

> Ye file kisi ko mat bhejna aur GitHub par upload mat karna. Isme aapke WiFi password hote hain.

<!-- ![Step 0B: Generate archive](screenshots/step-00b-backup.png) -->

---

## Step 1: Tailscale account aur phone app

1. Browser me `https://login.tailscale.com` kholo. **Sign up** karo (Google, Microsoft ya GitHub se).
2. Phone me **Tailscale** app install karo (App Store ya Play Store).
3. Phone me usi account se **login** karo, aur jab VPN permission maange to **Allow** karo.

**Kya dikhna chahiye:** `https://login.tailscale.com/admin/machines` par abhi aapka **phone** ek machine ke roop me dikhega (green dot ke saath).

<!-- ![Step 1: Machines page with phone](screenshots/step-01-machines-phone.png) -->

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

<!-- ![Step 2: Services menu with Tailscale](screenshots/step-02-services-tailscale.png) -->

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

<!-- ![Step 3: Login button](screenshots/step-03-login.png) -->
<img width="1722" height="928" alt="image" src="https://github.com/user-attachments/assets/c78a1978-d588-455c-aa38-55c79af44458" />

<!-- ![Step 3: Two machines green](screenshots/step-03-machines-green.png) -->
<img width="1597" height="617" alt="image" src="https://github.com/user-attachments/assets/817e6f5c-e311-4541-9507-7c44bfdb22f7" />

---

## Step 4: Key expiry band karo (nahi to router kuch mahine baad logout ho jayega)

1. `https://login.tailscale.com/admin/machines` kholo.
2. `openwrt` ke saamne wale **...** (teen dots) par click karo.
3. **Disable key expiry** dabao.

**Kya dikhna chahiye:** machine ke naam ke neeche **Expiry disabled** ka badge dikhta hai.

> Ye sirf **router** ke liye karo. Phone ka expiry on rehne do (Security wale hisse me wajah hai).But agar phone mein bhi karna chahte hai toh kar sakte hai.

<!-- ![Step 4: Disable key expiry](screenshots/step-04-disable-expiry.png) -->
<img width="650" height="630" alt="image" src="https://github.com/user-attachments/assets/952a44dd-6784-4a0c-86ad-a94ad4cf8934" />


---

## Step 5: Jio ODU ka IP pata karo

Zyadatar Jio ODU ka IP `192.168.225.1` hota hai, par apna pakka kar lo, kyunki ye alag ho sakta hai.

**Tareeka 1 (panel se):** LuCI me **Status → Overview** kholo. **Network** section me WAN ka **Gateway** dekho (agar dikhe). Wahi ODU ka IP hai.

**Tareeka 2 (SSH se):** ye command chalao:
```
ip route
```
Jo line `default via ...` se shuru hoti hai, uske saamne wala IP ODU ka hai. Jaise `default via 192.168.225.1 dev wan` me ODU ka IP `192.168.225.1` hai.

**Check karo:** laptop ko router ke WiFi se jodo aur browser me `https://<wo-IP>` kholo (jaise `https://192.168.225.1`). "Not secure" ka warning aayega, **Advanced → Proceed** dabao. **JioODU** wala login page (Jio ka logo aur username/password) dikhna chahiye. Dikhe to IP sahi hai.

> **Is guide me `<ODU-IP>` ka matlab:** aapke ODU ka IP. Agar aapka IP `192.168.225.1` hai, to jahan `<ODU-IP>/32` likha hai wahan `192.168.225.1/32` likho. Alag IP hai to wahi likho.

<!-- ![Step 5: Status Overview gateway](screenshots/step-05-gateway.png) -->

<!-- ![Step 5: JioODU login page](screenshots/step-05-odu-login.png) -->

---

## Step 6: Router ki Tailscale settings

**Services → Tailscale → General Settings** tab kholo.

> **Is page par sirf ek cheez badalni hai: "Advertise Routes".** Baaki settings zyadatar jaisi hain waisi rehni chahiye. Neeche ki table sirf check karne ke liye hai. Agar koi option samajh na aaye, use chhedo mat. (Poore setup ke liye Step 7 aur Step 8 bhi zaroori hain.)

### Advertise Routes (yahi asli kaam hai)
**Advertise Routes** me **do cheezein** daalo:
- Dropdown me se **`192.168.1.0/24`** chuno (ye router ka ghar wala network hai). Agar aapka router alag IP par hai (jaise `192.168.0.1`), to dropdown me jo aapka LAN dikhe wahi chuno.
- Dusre box me type karo **`192.168.225.1/32`** (apna ODU IP, `/32` ke saath) aur **Enter** dabao. Wo ek chip (chhota tag) ban jayega.

> **`/32` ka matlab sirf ek address.** Dropdown me `192.168.225.0/24` bhi dikhega, use **mat chunna**. Usse ODU ke peeche ke sab devices khul jate hain, jabki hume sirf ODU ka panel chahiye.

### Baaki settings (sirf check karo, badlo mat)

| Setting | Kya rakhna hai | Kyun |
|---|---|---|
| Firewall Mode | **nftables** | Aapke router ka firewall nftables hai |
| Accept Routes | band | Iski zaroorat nahi |
| Advertise Exit Node | band | Sirf optional Step 11 me chahiye |
| Allow LAN Access | band | Exit node ke liye hai |
| Enable Web Interface | band | Extra panel khulta hai, zaroorat nahi |
| Disable SNAT | **band** (tick mat karo) | Router bahar jaate traffic ka address badal deta hai, isse ODU ko jawab sahi jagah milta hai |
| Shields Up | **band** (tick mat karo) | Tick karne par bahar se aane wale connections rok diye ja sakte hain, isliye band rakho |
| Enable Tailscale SSH | band | Normal SSH hi chalega |
| Disable MagicDNS | band | Jaisa hai rehne do |
| Exit Node | **None** | Abhi nahi chahiye |

Neeche **Save & Apply** dabao.

**Kya dikhna chahiye:** Advertise Routes me do chip dikhte hain: `192.168.1.0/24` aur `192.168.225.1/32` (ya aapka ODU IP).

<!-- ![Step 6: Advertise routes chips](screenshots/step-06-advertise-routes.png) -->
<img width="717" height="842" alt="image" src="https://github.com/user-attachments/assets/efb23f4b-0d0b-4a5c-9d03-d87ce8d34aa1" />


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
aur Ta| Allow forward from source zones | **sirf lan** (wan mat jodo). Agar LAN ke devices se Tailscale devices tak jaana nahi hai to ise khali bhi chhod sakte ho |ilscale account ka password badal do (agar Google/Microsoft se login kiya hai to us account ka password).

5. Neeche **Save** dabao. Phir firewall page ke neeche **Save & Apply**.

> **Kyun `wan` chahiye?** Jio ODU router ke **WAN** side par hai. Tailscale se aaye traffic ko `wan` ki taraf bhejne ki permission na ho to router use aage nahi jaane deta. Ye ODU ka panel na khulne ki ek aam wajah ho sakti hai. "Source zones" me `wan` nahi jodna, nahi to ODU ki taraf se aapke Tailscale network me connection khul jayega.
>
> `wan` zone ki **Masquerading** pehle se on hoti hai. Use band mat karna.

**Command se karna ho (optional):** agar upar ki settings samajh na aaye, ya `wan` forward na dikhe, to SSH me ye chalao. Ye wahi cheez karta hai jo upar ki table me `wan` jodna karta hai. Dobara chalane se koi nuksaan nahi hota. Ye command tabhi chalao jab upar point 1 me **Auto Configure Firewall** dabake `tailscale` zone ban chuka ho.

```
uci set firewall.tailscale_to_wan='forwarding'
uci set firewall.tailscale_to_wan.src='tailscale'
uci set firewall.tailscale_to_wan.dest='wan'
uci commit firewall
/etc/init.d/firewall restart
```

**Kya dikhna chahiye:** firewall restart ke baad koi error nahi aata.

<!-- ![Step 7: tailscale zone settings](screenshots/step-07-firewall-zone.png) -->
<img width="1805" height="892" alt="image" src="https://github.com/user-attachments/assets/7780e61f-1e95-4e34-be1a-1a9100f6ff1c" />


---

## Step 8: Admin page me routes approve karo (ye step log sabse zyada bhoolte hain)

Router ne routes maange hain, par Tailscale tabhi use karta hai jab aap admin page par haan bolo.

1. `https://login.tailscale.com/admin/machines` kholo.
2. `openwrt` ke saamne **...** par click karo.
3. **Edit route settings...** dabao.
4. **Subnet routes** me dono routes **tick** karo:
   - `192.168.1.0/24`
   - `192.168.225.1/32` (ya aapka ODU IP)
5. **Save** dabao.

**Kya dikhna chahiye:** `openwrt` ke naam ke neeche **Subnets** ka badge (neela) aur **Expiry disabled** ka badge dikhte hain. Subnets na dikhe to page refresh karo.

<!-- ![Step 8: Edit route settings](screenshots/step-08-approve-routes.png) -->
<img width="717" height="842" alt="image" src="https://github.com/user-attachments/assets/250f49d3-7ebd-4884-873d-5d0ea6660842" />

---

## Step 9: Phone se test karo

1. Phone me **WiFi band** karo aur **mobile data on** karo. (Isse pakka hota hai ki aap sach me bahar se test kar rahe ho.)
2. **Tailscale app** kholo aur VPN **on** karo. App me router (`openwrt`) "Connected" dikhna chahiye.
3. Tailscale app ki settings me **Use Tailscale subnets** on hona chahiye (agar option dikhe).
4. Phone ke browser (Safari/Chrome) me kholo:

| Kya kholna hai | Address |
|---|---|
| Router ka panel | `http://192.168.1.1` |
| Jio ODU ka panel | `https://192.168.225.1` (apna ODU IP) |

**Kya dikhna chahiye:**
- `http://192.168.1.1` par **OpenWrt ka login page**.
- `https://192.168.225.1` par **JioODU ka login page** (Jio ka orange logo, username aur password).

> **"Not secure" ka warning:** Jio ODU ka certificate self-signed hai, isliye browser ek warning dega. Ye is apne device ke liye normal hai. Safari me **Show Details → visit this website**, Chrome me **Advanced → Proceed**. **Ye sirf is ODU ke liye karo, kisi aur website par nahi.**
>
> **Router ka doosra address:** router ka panel `http://<router-ka-100.x.x.x-IP>` se bhi khulta hai. Ye IP Tailscale app ya admin page me router ke saamne dikhta hai. Ye address route ke bina bhi kaam karta hai, to pehle ise try karo agar `192.168.1.1` na khule.

<!-- ![Step 9: Router panel on phone](screenshots/step-09-router-on-phone.png) -->
<!-- ![Step 9: JioODU panel on phone](screenshots/step-09-odu-on-phone.png) -->

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

## Maintenance: Tailscale ko update rakho

Tailscale ek VPN software hai, isliye use purana mat rehne do. Kuch mahine me ek baar router par SSH karke ye chalao (pehle config backup le lo):

```
apk update
apk upgrade tailscale luci-app-tailscale-community
```

Naya firmware flash karne ke baad router ka Tailscale login aksar chala jata hai. Aisa ho to **Step 3 dobara karo**, aur admin page par purani `openwrt` entry **...** → **Delete** kar do. Routes dobara approve karne pad sakte hain (Step 8).

---

## Dikkat aaye to (troubleshooting)

| Dikkat | Kya karo |
|---|---|
| Router me **Services → Tailscale** nahi dikhta | Step 2 karo, phir Ctrl + F5 |
| Status **LOGGED OUT** | Step 3 dobara karo. Naya firmware flash karne ke baad aksar aisa hota hai |
| Admin page me **Subnets** badge nahi | Step 8 karo, routes approve nahi hue |
| `192.168.1.1` bahar se nahi khulta | Phone me Tailscale ON hai? Mobile data par ho? Step 6 aur 8 check karo. Pehle `http://<router-ka-100.x.x.x-IP>` try karo |
| `192.168.1.1` khulta hai par **JioODU nahi** | Step 7 ke command wale forward rule ko chalao. Browser me `https://` use karo (`http` nahi). Laptop ke Chrome se bhi try karo |
| Jis network par ho wahan bhi `192.168.1.x` ya `192.168.225.x` hai | Browser wahan ke local device ko kholega, ghar wale ko nahi. Mobile data par ye dikkat nahi aati. Is case me `http://<router-ka-100.x.x.x-IP>` use karo |
| JioODU ka IP badal gaya | Step 5 se naya IP nikalo aur Step 6 aur 8 me naya `/32` daalo |
| `apk update` me **Operation not permitted** | 1-2 minute ruk ke dobara chalao. `ping -c 3 8.8.8.8` chalake internet check karo |
| `ssh` me **REMOTE HOST IDENTIFICATION HAS CHANGED** | PC ke PowerShell me `ssh-keygen -R 192.168.1.1` chalao, phir dobara SSH karo |
| Panel khulta hai par **dheere** | Tailscale kabhi-kabhi seedha connection nahi bana pata aur beech ke server (relay) se jata hai. Jio AirFiber par ye ho sakta hai. Router par `tailscale status` chalao. Phone ke saamne **relay** dikhe to yahi wajah hai. Kuch der baad **direct** ban sakta hai |

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

## Surakshit rehne ke liye (zaroor padho)

### Ye setup kya kholta hai
Is guide ke baad aapke **Tailscale network ka har device** router ka panel (LuCI aur SSH), aapke ghar ke LAN ke devices (`192.168.1.0/24`), aur Jio ODU ka panel kholne layak ho jata hai. Aapke akele ke tailnet me ye theek hai. Par isliye:

- Tailscale network me **sirf apne devices** rakho. **Apna tailnet ya router ka node kisi ko share mat karo.**
- **Phone chori ho jaye ya kho jaye** to turant admin page par us device ke **...** → **Delete** kar do, aur Tailscale account ka password badal do (agar Google/Microsoft se login kiya hai to us account ka password). Phone me screen lock aur biometric zaroor rakho.
- Phone ka **key expiry on rehne do** (sirf router ka band kiya hai). Phone kho jaye to uski login kuch mahine baad apne aap khatam ho jati hai.
- Zyada control chahiye (jaise "sirf ye phone sirf router ke port 443 tak pahunche") to Tailscale admin page ke **Access controls** me limit laga sakte ho. **Galat policy se aap khud lock ho sakte hain**, isliye pehle Tailscale ke docs padho aur default policy ka backup rakho.

### Baaki baatein
- Router ka **password strong** rakho. **Jio ODU ka default password badal do**, kyunki ab ye Tailscale se bhi pahunch me hai.
- **Router ka panel (LuCI) internet par kabhi expose mat karo** (port forwarding ya DMZ se). Bahar se sirf Tailscale ke raaste kholo.
- Sirf **`/32`** route advertise karo ODU ke liye. Poora subnet mat do.
- Router ka login link ya auth key kisi ko mat bhejo, aur kisi script ya GitHub me mat daalo.
- Router ka **config backup** (`.tar.gz`) aur agar partition backup liya ho to wo bhi **private** rakho. Inhe GitHub ya kisi public jagah par kabhi upload mat karo.
- Guide ya screenshots GitHub par daalne se pehle unme se apna **Tailscale IP, MAC address, serial number, WiFi password aur login link** hata do.

### Advanced (sirf experienced logon ke liye)
`192.168.1.x` bahut common subnet hai, isliye doosri jagah ke WiFi par kabhi-kabhi takrav hota hai. Chaho to router ka LAN subnet kuch anokha kar sakte ho (jaise `192.168.77.0/24`), par isse router ke saare settings aur devices badalte hain. **Aam logon ko ye karne ki zaroorat nahi hai**, aur galti se router tak pahunch kho sakte ho.

---

## Wapas hatana ho to

1. Router par: `tailscale down`
2. Admin page par `openwrt` ke **...** → **Delete**
3. LuCI → **Network → Firewall** me `tailscale` zone aur forwarding rule hata do
4. Advertise Routes ke dono chip hata do aur **Save & Apply** karo

---

## Aksar poochhe jane wale sawal

**Kya ye free hai?** Is guide ke likhe jaane ke time Tailscale ka personal plan free hai. Plan badal sakte hain, isliye Tailscale ka pricing page ek baar dekh lo.

**Kya Jio AirFiber par chalega, jahan public IP nahi hota?** Haan. Tailscale ko port-forwarding ya public IP nahi chahiye, kyunki connection aapke devices apne aap bana lete hain. Kabhi-kabhi relay se chalta hai aur speed kam ho sakti hai.

**Phone ki battery par asar?** Thoda hota hai. Zaroorat na ho to Tailscale app me VPN off kar do.

**Exit node zaroori hai?** Nahi. Sirf router aur ODU ke panel kholne ke liye Step 1-10 kaafi hain.

**Android par chalega?** Chalna chahiye, par abhi test nahi hua. Agar aap test karo to GitHub par issue ya pull request me batao.

---

## English summary

**What this does:** lets you open both your **OpenWrt router's panel** (`http://192.168.1.1`) and your **Jio ODU / AirFiber panel** (usually `https://192.168.225.1`) from anywhere using **Tailscale** (free), with no port forwarding and no public IP.

**How:** the router advertises two subnet routes to your tailnet, `192.168.1.0/24` and `<ODU-IP>/32` (just the single ODU address), you approve them in the Tailscale admin console, and you allow forwarding from the `tailscale` firewall zone to `wan` so traffic can reach the ODU.

**Steps in short:** set a router password and back up config, install Tailscale (`apk add tailscale luci-app-tailscale-community`), log in from LuCI (Services, Tailscale), disable key expiry for the router, find the ODU IP (`ip route`), set Advertise Routes, fix the firewall zone, approve routes in the admin console, test from a phone on mobile data, reboot and test again.

**Security notes:** every device in your tailnet can reach the router panel and your LAN, so keep the tailnet personal, set a strong router password, change the ODU default password, delete a lost phone from the admin console right away, and never upload backups or screenshots containing private data.

**Tested:** JioRouter AX6000 (JIDU6J01 / JIDU6401), OpenWrt 25.12.6, Tailscale 1.104, iPhone, October 2026. Android is untested. Unofficial community guide, use at your own risk.

---

## Credits

OpenWrt, Tailscale aur `luci-app-tailscale-community` ke developers ko dhanyavaad. Ye guide ek real JioRouter AX6000 par kiye gaye setup se bani hai.
