Bilal, yeh bohot hi aala sawaal hai! Jab hum pehli dafa AWS seekhte hain na, toh yeh baat sab ko confuse karti hai ke *"Bhai jab Internet Gateway (IGW) bana diya hai aur firewall (Security Group) bhi hai, toh beech mein yeh Route Table ka kya kaam?"*

Chalein in donon ka farq bilkul aasan, aam zindagi ki misaal se samajhte hain.

---

### 🗺️ Route Table Kya Hai aur Iska Maqsad Kya Hai?

Internet Gateway (IGW) sirf ek **Darwaza (Gate)** hai. Lekin aapki VPC ke andar jo subnets hain, unhein kaise pata chalega ke bahar jaane ke liye us darwaze tak kaise poonchna hai?

**Route Table asal mein ek Road Signboard (Rasta batane wala nishaan) hai.**

Imagine karein aap ek bohot bari building (VPC) mein hain jismein bohot se kamre (Subnets) hain. Building ka ek main gate (IGW) hai jo baahar road par khulta hai.

* Agar kamre ke baahar koi signboard (Route Table) nahi laga hoga jismein likha ho *"Bahar jaane ke liye Main Gate ki taraf jao"*, toh kamre ke andar baitha banda bhatakta rahega aur baahar nahi nikal sakega.
* **Route Table mein hum yeh rule likhte hain:** `0.0.0.0/0` (yani internet par kahin bhi jana ho) -> `igw-XXXXX` (toh is Internet Gateway ka rasta pakro).

**Public aur Private ka farq hi Route Table se banta hai:**

* Jis Subnet ke Route Table par IGW ka signboard laga ho, woh **Public Subnet** ban jata hai.
* Jis Subnet ke Route Table par IGW ka signboard NA laga ho (ya sirf NAT Gateway ka laga ho), woh **Private Subnet** ban jata hai.

---

### 🛡️ Security Group Kya Hai aur Iska Maqsad Kya Hai?

Route Table ne rasta toh bata diya, lekin raste par aane jaane wale logon ki cheking kaun karega?

**Security Group aapka Security Guard (Firewall) hai.**

Yeh bilkul aapke EC2 instance (Server) ke darwaze par baithta hai aur har aane jaane wali request ka ID card check karta hai. By default, AWS har aane wali cheez ko **Block** rakhta hai. Aapko khud guard ko batana parta hai ke kisko andar aane dena hai.

* **Port 22 (SSH):** Aap guard (Security Group) ko bolte ho: *"Agar koi black terminal se SSH karne aaye, toh use andar aane do."*
* **Port 80 (HTTP):** *"Agar koi browser se website dekhne aaye, toh use aane do."*
* **Source Restriction:** Aap guard ko yeh bhi bol sakte hain ke *"Sirf mere ghar ke IP se ya sirf mere Public Server se traffic andar aane dena, baaki kisi hacker ko mat ghusne dena."*

---

### 📊 Quick Summary (Dono ka farq ek sath)

| Feature | 🗺️ Route Table (RT) | 🛡️ Security Group (SG) |
| --- | --- | --- |
| **Asli Kaam** | **Rasta Dikhana (Routing)** | **Hifazat Karna (Security)** |
| **Misaal** | Google Maps ya Road Signboard | Bandook wala Security Guard |
| **Sawaal** | *"Mujhe internet par jana hai, main kis raste se jaon?"* | *"Kya is bande ke paas andar aane ki permission hai?"* |
| **Kahan Lagta Hai** | Yeh poore **Subnet** par lagta hai. | Yeh specific **EC2 Instance** par lagta hai. |

Ab samajh aya? Route Table rasta banata hai aur Internet Gateway tak le kar jata hai, jabke Security Group server ke darwaze par pehra deta hai taake koi ghalat banda andar na ghuse.

Kya ab yeh donon concepts clear hain Bilal?
