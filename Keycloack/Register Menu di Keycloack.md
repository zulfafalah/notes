
Untuk meregister menu di keycloack ada beberapa hal yang perlu di buat 

## 1. Membuat Scope
Cara membuat Scope
1. Pergi ke client (test-pricate-client)
2. Pergi ke Authorization -> Scope, kemudian klik **Create Authorization Scope**
![[Pasted image 20251119112743.png]]

3. Isi form nya untuk membuat scope
![[Pasted image 20251119113108.png]]
4. Done


## 2. Membuat Resource 
Cara membuat Resource
1. Membuat resource di client (test-pricate-client)

![[Pasted image 20251119111603.png]]

2. Kemudian pergi tab Authorization -> Resource
![[Pasted image 20251119112049.png]]

3. isi form nya untuk create resource
![[Pasted image 20251119112237.png]]

![[Pasted image 20251119112454.png]]
4. Isi Authorization Scopes yang di buat tadi 
![[Pasted image 20251119114324.png]]
5. Done

## 3. Membuat Roles
Cara membuat role sebagai berikut:
1.  Pergi ke client (test-pricate-client) kemudan ke tab Role
![[Pasted image 20251119114531.png]]
2. Buat role dengan menekan tombol **Create Role**
![[Pasted image 20251119114626.png]]
3. Done

## 4. Membuat Policies
Cara membuat policies sebagai berikut:
1. Pergi ke client (test-pricate-client) kemudian ke tab Authorization dan klik **Create client policy**
![[Pasted image 20251119115127.png]]
2. Setelah klik *create client policy* kemudian pilih **Role** 
![[Pasted image 20251119115428.png]]
3. Isi form untuk membuat role nya dan set role fetch nya menjadi true 
![[Pasted image 20251119115627.png]]
4. Kemudian masukan roles kita buat tadi
![[Pasted image 20251119120221.png]]
5. Pilih Role yang kta bikin tadi 
![[Pasted image 20251119120451.png]]
6. Save dan selesai

## 5. Membuat Permission
Cara untuk membuat permission
1. Pergi ke client (test-pricate-client) kemudian ke tab Authorization dan klik **Permission** dan klik *Create Permission* kemudian pilih **Create resource-based-permission**
![[Pasted image 20251119120739.png]]

2. Isi form permission nya, isi Resource sesuai say kita bikin di atas
![[Pasted image 20251119120905.png]]

3. Kemudian isi juga policies nya dengan Policies yang kita bikin diatas
![[Pasted image 20251119121035.png]]
4. Kemudian Save
5. Done

## 6. Assign Role ke User/ Group
Cara untuk assign ke user:
1. Pergi ke Users kemudian search user yang ingin ditambahkan role nya
2. Pergi ke tab Role mappin kemudian cari role yang kita bikin diatas 
![[Pasted image 20251119122129.png]]
3. Done
