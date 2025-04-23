# Konfiguracja klastra Proxmox

---

---

Jesteś administratorem sieci w firmie X.

Otrzymał(a)eś od przełożonego prośbę, aby mając do dyspozycji 3 nowe serwery (VM1, VM2, VM3), wdrożyć wysoko dostępny klaster dla maszyn wirtualnych zwirtualizowanego środowiska XCP-ng lub Proxmox-VE.

Otrzymałeś również poniższe założenia, jakie mają zostać spełnione dla klastra jaki masz wdrożyć:

1. Klaster ma zostać wdrożony jako klaster wysokodostępny z wykorzystaniem trzech
węzłów. Ma on również wykorzystywać na potrzeby realizacji wysokiej dostępności współdzielony zasób dyskowy w postaci współdzielonej jednostki LUN w ramach udostępnionego celu iSCSI (uruchomionego na osobnym jakimś wybranym systemie). W gotowym klastrze należy utworzyć dwie wirtualne maszyny z systemem Windows Server 2022, i przetestować na nich prawidłowe działanie klastra i mechanizmów wysokiej dostępności.
2. Wdrożony klaster wysokodostępny powinien mieć skonfigurowane połączenie do lokalnej sieci komputerowej na takiej zasadzie, że powinny istnieć dwa osobne połączenia do dwóch różnych przełączników Cisco Catalyst, działających na takiej zasadzie, że gdy jedno podstawowe połączenie ulegnie awarii, to drugie przejmie jego rolę. Ponadto biorąc pod uwagę, że w firmie są wykorzystywane VLAN'y (VLAN 20, oraz VLAN 30), to klaster środowiska zwirtualizowanego ma obsługiwać te VLAN'y, w taki sposób iż jeden z Windows Server ma być przyłączony do VLAN20, a drugi do VLAN 30.
3. Wdrożyć wysokodostępny cel iSCSI, bazujący na systemie operacyjnym GNU/Linux, na którym będzie udostępniona jednostka LUN wykorzystywana jako współdzielony zasób dyskowy przez utworzony wcześniej klaster środowiska zwirtualizowanego. Tenże klaster pracy awaryjnej iSCSI ma działać na takiej zasadzie, że składać się ma z dwóch węzłów, synchronizujących dane między sobą w locie, natomiast sama usługa iSCSI ma zostać udostępniona jako klaster Active<>Passive (jeżeli jeden węzeł ulegnie awarii, to drugi posiadający wszystkie dane, przejmie jego rolę).

---

# Konfiguracja Klastra Proxmox

Po instalacji proxmoxa na 3 maszynach (każda z nich posiada 2 interfejsy sieciowe), można połączyć się do panelu webowego za pomocą ip serwera.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled.png)

Domyślnie proxmox korzysta z samopodpisanego certyfikatu ssl. Dlatego widnieje ostrzerzenie o niezabezpieczonym połączeniu.

## Tworzenie klastra

Przechodząc w panelu webowym do zakładki “Klaster”, można utworzyć klaster. Korzystając z wygenerowanych danych możemy przyłączyć pozostałe maszyny do utworzonego klastra.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%201.png)

## Tworzenie maszyn wirtualnych

Utworzenie przykładowej maszyny z systemem windows.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%202.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%203.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%204.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%205.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%206.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%207.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%208.png)

podczas tworzenia maszyn można wybrać VLAN do którego ma należeć maszyna:

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%209.png)

Po zakończeniu konfiguracji można przejść do instalacji systemu. Na obu maszynach zainstalowałem windows server core 2019. Dodatkowo pierwsza maszyna (100) jest w VLANie 20, a druga (101) w VLANie 30.

(Utworzenie celu iSCSI i migracja zasobów vm w ostatniej sekcji)

# Konfiguracja sieci w GNS3

Utworzona konfiguracja sieci w gns3. Zgodnie z treścią zadania.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2010.png)

Jednakże, w normalnych warunkach, raczej powinien istnieć odrębny segment fizyczny sieci wyznaczony dla iSCSI. Przykładowa konfiguracja:

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2011.png)

W takim przypadku klaster z celem iSCSI, jest odłączony od sieci lokalnej. Można też dodać po dwa interfejsy sieciowe do węzłów klastra i podłączyć je do przełączników sieci lokalnej.

## Bonding połączeń sieciowych w proxmox

Połączenie dwóch interfejsów sieciowych w trybie active-backup, czyli komunikacja odbywa się tylko na jednym z nich, a drugi funkcjonuje jako intefejs awaryjny.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2012.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2013.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2014.png)

Trzeba pamiętać o utworzeniu takiej konfiguracji na każdym z węzłów klastra.

## Konfiguracja VLAN na przełącznikach

Informacje, nt. VLAN:

[https://www.youtube.com/watch?v=cjFzOnm6u1g&ab_channel=Jeremy'sITLab](https://www.youtube.com/watch?v=cjFzOnm6u1g&ab_channel=Jeremy%27sITLab)

[https://www.youtube.com/watch?v=Jl9OOzNaBDU&ab_channel=Jeremy'sITLab](https://www.youtube.com/watch?v=Jl9OOzNaBDU&ab_channel=Jeremy%27sITLab)

[https://www.youtube.com/watch?v=OkPB028l2eE&ab_channel=Jeremy'sITLab](https://www.youtube.com/watch?v=OkPB028l2eE&ab_channel=Jeremy%27sITLab)

trunk porty do klastra proxmox. dla testów można ustawić jakieś VPCs i do nich przypisać access porty z określonym VLANem.

# Konfiguracja klastra Active - Passive

Na dwóch systemach debian11 instalacja drdb i pacemaker w celu utworzenia 

Aby utworzyć wysokodostępny cel iSCSI należy najpierw utworzyć i zsynchronizować nośnik DRBD między dwoma linuxami (do tego celu wykorzystałem dodatkowe dyski 80GB każdy, dołączone do obu maszyn).

Następnie trzeba utworzyć klaster z obu maszyn, do tego celu wykorzystałem pacemaker.

Przy takiej konfiguracji można utworzyć wysokodostępny cel iSCSI.

## DRBD

Konfiguracja utworzona na podstawie Pana filmów oraz artykułu z linux journal. Odnośniki w ostatniej sekcji.

Pamiętać o utworzeniu wpisów do /etc/hosts dla obu linuxów.

Konfiguracja zasobu

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2015.png)

Pierwsza synchronizacja

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2016.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2017.png)

## Pacemaker

Instalacja i konfiguracja pacemaker-a na podstawie: [https://clusterlabs.org/pacemaker/doc/deprecated/en-US/Pacemaker/2.0/html/Clusters_from_Scratch/index.html](https://clusterlabs.org/pacemaker/doc/deprecated/en-US/Pacemaker/2.0/html/Clusters_from_Scratch/index.html)

Na samym starcie wywołanie komendy `#pcs cluster destroy`, aby łatwiej było skonfigurować nowy klaster.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2018.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2019.png)

Będzie potrzebne ponowne uruchomienie corosync i pacemaker. po uruchomieniu klastra:

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2020.png)

Dodanie ip klastra

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2021.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2022.png)

Teoretycznie już mamy klaster typu Active - Passive.

Dodanie drbd jako zasobu klastra.

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2023.png)

Pamiętać o połączeniu interfejsów sieciowych jak na maszynach “proxmox”

# Wysoka dostępność VM

### iSCSI

Udostępnienie zasobu iSCSI:

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2024.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2025.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2026.png)

Pobranie wszystkich iqn inicjatora z pliku `/etc/iscsi/initiatorname.iscsi`, i przekazanie ich do listy acl

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2027.png)

### Przyłączenie zasobu iSCSI

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2028.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2029.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2030.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2031.png)

### Migracja dysku na zasób iSCSI

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2032.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2033.png)

Trzeba pamiętać jeszcze o dodaniu tych maszyn do sekcji “HA”

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2034.png)

## Testowa awaria

Wyłączenie proxmox3 w trakcie migracji maszyny. 

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2035.png)

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2036.png)

Stan już po chwili

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2037.png)

Działa

![Untitled](Konfiguracja%20klastra%20Proxmox%20838f50aa13794a848c58c27bb30a9229/Untitled%2038.png)

---
