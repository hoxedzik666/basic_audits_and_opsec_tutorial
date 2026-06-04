# Wstepu slowami...

**Siemanko to znowu ja hah w tej serii tutoriali uczyc bedziemy sie przedewszystkim oczywiscie "etycznych" metod atakow na swoje wytwory np aplikacje webowe, co bardziej nazwalibysmy Audytem bezpieczenstwa. I wlasnie od tego procesu sobie zaczniemy z racji ze w tych tutorialach bedziemy chcieli skupic sie przedewszystkim na efektywnosci oraz efekcie takze finalnie dojdziemy do etapu gdzie glownie bedziemy promptowac do gemini ale kurwa spokojnie wariacie pierw nauczymy sie podstawowych narzedzi do audytyowania z palca....**

### Czemu AI bedzie wazne wgl w tej serii? 

==**odpowiadajac na to bardzo glupie pytanie... Mimo ze nie bardzo mam ochote XD.**== 

- ***Czemu macie byc gorsi i wolniejsi od reszty preznie rozwijajacego sie rynku agentow i modeli AI*** 

**Takze dwa w jednym bo nauczymy sie odrazu fajnie promptowac do (na start gemini.)** 

**Oczywiscie nie musze tlumaczyc ze bede korzystal tylko i wylacznie z jakiegos systemu z mojej ukochanej Linuxowej rodziny...** 

***Zdaje sobie ze jestem w mniejszosci jesli chodzi o dobor OS ale czy mi z tym jakos zle?*** 

- *na to pytanie juz sobie pozwole nie odpowiadac :P*

> na ten moment jako podstawka mam zainstalowanego debiana 13, jesli chcesz wgraj sobie kali linuxa czy jaki tam ci os z rodziny Linuxowatych odpowiada....
> 
> 	- pamietaj jednak ze moim pakiet menagerem bedzie **APT** wiec jesli wgrasz sobie np *arch'a* bedziesz musial korzystac z pacmana przy instalacji programow czy aktualizacji systemu etc etc. 
> 		- *jesli sobie pereleczko wgrasz kaliego ktory jest oparty na debianie twoim pakiet menagerem rowniez bedzie apt.... mowie to tylko jesli zdazyles zapomniec bo przeciez na tym etapie juz o tym wiedziales prawda?*


**Dobra jednak dla those co wybrali ubuntu, debiania czy jaka kolwiek inna dystrybucje nie przeznaczona domyslnie dla pentesterow podaje narzedzia ktore bedziemy instalowac w celu pozniejszego uzycia..** 

Dobra wiec tak panie monter wypadalo by zaczac.... 

> 	-od aktualizacji? 
> 	-od wgrania mojej ulubionej przegladarki? 
> 

*Taki chuj za przeproszeniem w wiekszosci przypadkow wypadalo by zaczac od modyfikacji pliku sudoers a mowie o tym teraz zebys zaraz nie grzebal w googlach i sie nie wkurwial :P* 

```bash
sudo nano /etc/sudoers

#podaj haslo do konta....
```

*Teraz znajdz linijke*

```plain text

# User privilege specification
root    ALL=(ALL:ALL) ALL

```

i pod spodem dopisz 

```plain text

# User privilege specification
root    ALL=(ALL:ALL) ALL
nazwa_twojego_konta    ALL=(ALL:ALL) ALL

```

==***Zapisujesz uzywajac kombinacji CTR + X  / y / enter :)***== 

**Okej teraz finalnie mozesz przejsc do aktualizacji pakiet menagera wpisujac**

```bash

sudo apt update && sudo apt full-upgrade -y 

#haslo do konta administratora 
#teraz utworzymy sobie skrypt bash za pomoca nano ktory zainstaluje nam wszystkie narzedzia zacznijmy od stworzenia folderu 

cd /home/$USER/Dokumenty #lub Documents 
&& mkdir scripts_sh && cd scripts_sh && nano tools.sh 

#now wklej to co masz pod spodem do pliku tools.sh zapisz go a nastepnie postepuj zgodnie z instrukcja. 
```

Oto kompletny skrypt instalacyjny w języku Bash dla systemu **Debian 13 (Trixie)**, korzystający z menedżera pakietów `apt`. 

Skrypt automatycznie instaluje wszystkie wymagane środowiska uruchomieniowe (Java JRE dla DirBuster, Ruby dla WhatWeb, Perl dla Nikto, Python dla sqlmap/dirsearch oraz Go dla Nuclei), instaluje narzędzia, uruchamia i konfiguruje usługę Tor oraz paruje ją z Proxychains4.

```bash
#!/bin/bash

# Skrypt instalacyjny narzędzi bezpieczeństwa dla systemu Debian 13 (Trixie)
# Uwzględnia: nmap, sqlmap, nikto, nuclei, whatweb, dirbuster, ffuf, dirsearch, tor, proxychains4

# Przerwij wykonywanie skryptu w przypadku błędu dowolnej komendy
set -e

echo "[+] Aktualizacja list pakietów systemowych..."
sudo apt update

echo "[+] Instalacja wymaganych zależności oraz środowisk uruchomieniowych..."
# default-jre  -> dla DirBuster (Java)
# python3, python3-pip, python3-venv -> dla sqlmap, dirsearch
# ruby         -> dla WhatWeb
# perl         -> dla Nikto
# golang-go    -> do kompilacji i instalacji Nuclei
# git, curl, wget, unzip -> narzędzia pomocnicze do pobierania zasobów
sudo apt install -y \
    apt-transport-https \
    curl \
    wget \
    git \
    unzip \
    python3 \
    python3-pip \
    python3-venv \
    default-jre \
    ruby \
    perl \
    golang-go

echo "[+] Instalacja narzędzi dostępnych bezpośrednio w reporzytoriach apt..."
# Uwaga: Narzędzie 'ffuz' z Twojego zapytania to w rzeczywistości 'ffuf' (Fuzz Faster U Fool)
sudo apt install -y \
    nmap \
    sqlmap \
    nikto \
    whatweb \
    dirbuster \
    ffuf \
    dirsearch \
    tor \
    proxychains4

echo "[+] Instalacja i kompilacja Nuclei za pomocą środowiska Go..."
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Instalacja najnowszej wersji Nuclei bezpośrednio z oficjalnego repozytorium Go
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Kopiowanie binarki do lokalizacji systemowej w celu globalnego dostępu
if [ -f "$HOME/go/bin/nuclei" ]; then
    sudo cp "$HOME/go/bin/nuclei" /usr/local/bin/
    echo "[+] Nuclei zainstalowane pomyślnie w /usr/local/bin/nuclei"
else
    echo "[!] Go install nie powiodło się. Próba pobrania prekompilowanej binarki z GitHub..."
    ARCH=$(uname -m)
    if [ "$ARCH" = "x86_64" ]; then ARCH_SUFFIX="amd64"; else ARCH_SUFFIX="386"; fi
    LATEST_NUCLEI=$(curl -s https://api.github.com/repos/projectdiscovery/nuclei/releases/latest | grep -oP '"browser_download_url": "\K[^"]+linux_'"$ARCH_SUFFIX"'\.zip' | head -n 1)
    wget -O nuclei.zip "$LATEST_NUCLEI"
    unzip -o nuclei.zip nuclei
    sudo mv nuclei /usr/local/bin/
    rm -f nuclei.zip
fi

echo "[+] Uruchamianie i włączanie autostartu usługi Tor..."
sudo systemctl enable tor
sudo systemctl start tor

echo "[+] Konfiguracja Proxychains4 do współpracy z siecią Tor..."
# Tor domyślnie nasłuchuje jako proxy SOCKS5 na porcie 9050.
# Konfigurujemy proxychains4, aby kierował ruch przez ten port.
if [ -f /etc/proxychains4.conf ]; then
    # Usuwamy domyślne/stare wpisy socks4, jeśli istnieją, i dodajemy aktualny SOCKS5 dla Tora
    if ! grep -q "socks5[[:space:]]*127.0.0.1[[:space:]]*9050" /etc/proxychains4.conf; then
        echo "socks5 127.0.0.1 9050" | sudo tee -a /etc/proxychains4.conf > /dev/null
        echo "[+] Dodano konfigurację SOCKS5 (Tor) do /etc/proxychains4.conf"
    fi
else
    echo "[!] Nie znaleziono pliku /etc/proxychains4.conf. Upewnij się, czy pakiet zainstalował się poprawnie."
fi

echo -e "\n[+] WERYFIKACJA INSTALACJI NARZĘDZI:"
echo "--------------------------------------------------"
nmap --version | head -n 1 || echo "[-] Nmap: Błąd"
sqlmap --version | head -n 1 || echo "[-] Sqlmap: Błąd"
nikto -Version | head -n 1 || echo "[-] Nikto: Błąd"
nuclei -version | head -n 1 || echo "[-] Nuclei: Błąd"
whatweb --version | head -n 1 || echo "[-] Whatweb: Błąd"
ffuf -V || echo "[-] ffuf: Błąd"
dirsearch --version | head -n 1 || echo "[-] Dirsearch: Błąd"
tor --version | head -n 1 || echo "[-] Tor: Błąd"
proxychains4 -h 2>&1 | head -n 1 || echo "[-] Proxychains4: Błąd"
echo "[-] DirBuster (wymaga środowiska graficznego X11/Wayland do uruchomienia GUI)"
echo "--------------------------------------------------"
echo "[+] Instalacja zakończona sukcesem!"
```

### Jak uruchomić skrypt:
1. Zapisz powyższy kod do pliku, np. `install_tools.sh`.
2. Nadaj mu uprawnienia do wykonywania:
   ```bash
   chmod +x install_tools.sh
   ```
3. Uruchom skrypt (wymagane uprawnienia `sudo` zostaną wywołane wewnątrz skryptu):
   ```bash
   ./install_tools.sh
   ```

**Okej zainstalujmy jeszcze Anonsurfa moze aler nie musi sie kiedys przydac tak?***

---

## Instalacja i obsługa Debian Anonsurf (Port ParrotSec Anonsurf)

**Anonsurf** to potężne narzędzie przeniesione z dystrybucji ParrotSec, które pozwala na anonimizację całego ruchu sieciowego w systemie za pomocą sieci TOR oraz reguł `iptables`. Dodatkowo pakiet zawiera moduł **Pandora**, który dba o czyszczenie pamięci RAM.

### Opis tego repozytorium
Ta wersja łączy w sobie pakiety `anonsurf` oraz `pandora` od ParrotSec w jedną całość, wprowadzając kilka istotnych poprawek:
* Korzysta z serwerów DNS od **Private Internet Access** (zamiast FrozenDNS).
* Zawiera poprawki dla użytkowników, którzy nie korzystają z aplikacji `resolvconf`.
* Usunięto zbędne funkcjonalności, takie jak interfejs graficzny (GUI) oraz uruchamianie przeglądarki Iceweasel w pamięci RAM.

---

### 1. Instalacja

Instalacja jest niezwykle prosta, ponieważ repozytorium zawiera gotowy skrypt instalacyjny. Aby zainstalować narzędzie na swoim systemie Debian, wykonaj następujące polecenia w terminalu:

```bash
# Klonowanie oficjalnego repozytorium
git clone https://github.com/anoopmsivadas/debian-anonsurf.git

# Przejście do katalogu z pobranym projektem
cd debian-anonsurf

# Nadanie uprawnień do uruchomienia instalatora
chmod +x installer.sh

# Uruchomienie instalatora z uprawnieniami superużytkownika (root)
sudo ./installer.sh
```

Po zakończeniu działania instalatora oba moduły (`anonsurf` oraz `pandora`) będą w pełni gotowe do użycia.

---

### 2. Korzystanie z Pandora (Czyszczenie RAMu)

**Pandora** automatycznie nadpisuje i oczyszcza pamięć RAM podczas wyłączania systemu, aby zapobiec odzyskaniu wrażliwych danych.

Możesz ją również wywołać ręcznie w dowolnym momencie:
```bash
sudo pandora bomb
```
> ⚠️ **UWAGA:** Uruchomienie tego polecenia natychmiast wyczyści całą pamięć podręczną systemu, co spowoduje m.in. zerwanie aktywnych tuneli i sesji SSH.

---

### 3. Korzystanie z Anonsurf

**Anonsurf** przekierowuje cały ruch systemowy przez sieć TOR przy użyciu reguł IPTables.

> ⚠️ **WAŻNA UWAGA:** **NIE** uruchamiaj tego narzędzia jako usługi systemowej (np. `sudo service anonsurf start`). Zamiast tego uruchamiaj je bezpośrednio za pomocą komendy `sudo anonsurf $COMMAND`.

#### Podstawowe komendy:

* **Uruchomienie anonimizacji:**
  ```bash
  sudo anonsurf start
  ```
  *Uruchamia systemowe tunelowanie ruchu przez proxy TOR za pomocą reguł iptables.*

* **Wyłączenie anonimizacji:**
  ```bash
  sudo anonsurf stop
  ```
  *Przywraca pierwotne ustawienia iptables i powraca do czystego, bezpośredniego połączenia z internetem.*

* **Restart usługi:**
  ```bash
  sudo anonsurf restart
  ```
  *Wykonuje kolejno procedurę zatrzymania ("stop") i ponownego uruchomienia ("start").*

* **Zmiana tożsamości (IP):**
  ```bash
  sudo anonsurf change
  ```
  *Zmienia Twoją tożsamość w sieci poprzez restart usługi TOR i pobranie nowego adresu IP.*

* **Sprawdzenie statusu:**
  ```bash
  sudo anonsurf status
  ```
  *Sprawdza, czy AnonSurf działa prawidłowo i czy Twój ruch jest bezpiecznie tunelowany.*

#### Funkcje powiązane z siecią I2P:

* **Uruchomienie usług I2P:**
  ```bash
  sudo anonsurf starti2p
  ```
* **Zatrzymanie usług I2P:**
  ```bash
  sudo anonsurf stopi2p
  ```

### 4. Przetestujmy to... 

**Okej teraz po wpisaniu w terminal**

```bash
anonsurf
```

![[anonsurf.png]]

***Powinnismy dostac taki efekt jak go wystartowac masz opisane wyzej takze odpuszcze sobie, ale przy okazji pokaze jak przekierowac ruch przegladarki firefox przez siec TOR. < tak wiem ze anonsurf przekierowywuje caly ruch siecii >*** 

---

## Zaawansowana konfiguracja przeglądarki Firefox pod kątem prywatności i anonimowości

Chociaż narzędzia systemowe (jak Anonsurf) przekierowują ruch na poziomie systemu operacyjnego, sama przeglądarka internetowa bez odpowiedniej konfiguracji i wtyczek jest gigantycznym źródłem wycieków danych. Poniżej znajduje się kompletny przewodnik, jak zamienić standardowego Firefoksa w pancerną przeglądarkę chroniącą Twoją tożsamość.

---

### 1. Ręczne przekierowanie ruchu przez sieć Tor (Proxy SOCKS5)

Jeśli nie chcesz tunelować całego systemu, a jedynie ruch z samej przeglądarki Firefox, możesz skonfigurować ją tak, aby bezpośrednio korzystała z lokalnego klienta sieci Tor (który domyślnie nasłuchuje na porcie `9050` jako proxy SOCKS5):

1. Otwórz menu Firefox i wejdź w **Ustawienia** (Settings) lub wpisz `about:preferences` w pasku adresu.
2. Na karcie **Ogólne** (General) zjedź na sam dół do sekcji **Ustawienia sieciowe** (Network Settings) i kliknij przycisk **Ustawienia...** (Settings...).
3. Zaznacz opcję **Ręczna konfiguracja serwerów proxy** (Manual proxy configuration).
4. Znajdź pole **Host SOCKS** (SOCKS Host) i wpisz tam adres IP: `127.0.0.1` oraz port: `9050`.
5. Upewnij się, że poniżej zaznaczona jest opcja **SOCKS v5**.
6. **KLUCZOWY KROK:** Zaznacz pole **Przekierowuj zapytania DNS przy używaniu SOCKS v5** (Proxy DNS when using SOCKS v5). Jeśli tego nie zrobisz, Twoja przeglądarka będzie wysyłać zapytania DNS poza siecią Tor, co doprowadzi do natychmiastowego wycieku Twoich zapytań!
7. Kliknij **OK**, aby zapisać zmiany.

---

### 2. Wyłączanie WebRTC w Firefox

**WebRTC (Web Real-Time Communication)** to technologia wbudowana w przeglądarki, która umożliwia bezpośrednią komunikację audio/wideo oraz przesyłanie danych P2P (np. na Discordzie, Zoomie czy Google Meet) bez pośrednictwa serwerów. 

Mimo że jest bardzo przydatna, stanowi **ogromne zagrożenie dla prywatności**, ponieważ potrafi ominąć proxy, VPN, a nawet sieć Tor, odpytując Twój system bezpośrednio o lokalny i zewnętrzny adres IP za pomocą protokołu STUN.

#### Jak całkowicie wyłączyć WebRTC w Firefox:
1. Wpisz w pasku adresu przeglądarki `about:config` i naciśnij Enter.
2. Zaakceptuj ostrzeżenie o ryzyku, klikając **Akceptuję ryzyko, kontynuuj** (Accept the Risk and Continue).
3. W polu wyszukiwania wpisz frazę: `media.peerconnection.enabled`.
4. Kliknij dwukrotnie na znalezioną pozycję (lub użyj przycisku przełącznika po prawej stronie), aby zmienić jej wartość z **true** na **false**.
5. Od tej pory WebRTC jest całkowicie zablokowane i nie ujawni Twojego prawdziwego IP.

---

### 3. Niezbędne wtyczki (Add-ons) zwiększające prywatność

Aby zainstalować poniższe dodatki, przejdź do oficjalnego sklepu z rozszerzeniami: **Firefox Add-ons** (`about:addons` -> sekcja "Znajdź więcej dodatków" na samym dole).

Oto lista i krótki opis najważniejszych wtyczek, które musisz wdrożyć:

*   **NoScript Security Suite (NoJS)**
    *   *Opis:* Blokuje wykonywanie wszelkich skryptów JavaScript, Java, Flash i innych aktywnych zawartości na stronach internetowych. JavaScript to najpotężniejsza broń w rękach systemów śledzących oraz hakerów (umożliwia profilowanie sprzętowe, exploitowanie przeglądarki czy odczyt parametrów systemu). NoScript pozwala na precyzyjne włączanie skryptów tylko dla zaufanych domen (biała lista).
*   **CanvasBlocker**
    *   *Opis:* Chroni przed zaawansowaną metodą śledzenia zwaną *Canvas Fingerprinting* (odciskiem palca płótna HTML5). Zamiast całkowicie blokować elementy graficzne (co mogłoby popsuć wygląd stron), CanvasBlocker generuje fałszywy szum lub losowe dane przy próbie odczytu unikalnych właściwości Twojej karty graficznej i silnika renderującego. Dzięki temu dla każdego trackera wyglądasz jak zupełnie nowy użytkownik.
*   **Privacy Badger**
    *   *Opis:* Dodatek stworzony przez fundację EFF (Electronic Frontier Foundation). Nie opiera się na prostych, statycznych listach blokowania. Zamiast tego analizuje zachowanie skryptów na odwiedzanych stronach. Jeśli wykryje, że dany skrypt śledzi Cię na różnych witrynach bez Twojej zgody, Privacy Badger automatycznie zablokuje mu możliwość wysyłania i odbierania danych.
*   **User-Agent Switcher and Manager (User Agent Controller)**
    *   *Opis:* Pozwala na łatwe fałszowanie nagłówka `User-Agent`. Nagłówek ten informuje serwer o tym, z jakiego systemu operacyjnego (np. Linux, Windows, macOS) oraz z jakiej przeglądarki korzystasz. Dzięki tej wtyczce możesz udawać, że przeglądasz internet z telefonu z Androidem lub iPhona, co utrudnia powiązanie Twoich sesji i chroni przed profilowaniem.
*   **Nuke Anything / Nuke Data**
    *   *Opis:* Pozwala na usuwanie dowolnych elementów ze strony internetowej (np. denerwujących popupów, skryptów blokujących AdBlocka, formularzy logowania czy elementów śledzących) bezpośrednio z menu kontekstowego (pod prawym przyciskiem myszy). Dodatkowo ułatwia szybkie niszczenie danych sesyjnych powiązanych z wybranym elementem lub całą witryną.
*   **Clear Cache (np. Clear Cache Button)**
    *   *Opis:* Dodaje prosty przycisk na pasku narzędzi przeglądarki, który pozwala jednym kliknięciem całkowicie wyczyścić pamięć podręczną (cache) przeglądarki. Cache przechowuje pliki stron internetowych, co może być wykorzystane do tzw. "cache trackingu" (identyfikacji użytkownika na podstawie zapisanych wcześniej plików graficznych czy skryptów).
*   **Cookie Quick Manager**
    *   *Opis:* Zaawansowane narzędzie do pełnej kontroli nad plikami cookies (ciasteczkami). Umożliwia podgląd, edycję, usuwanie, a także eksportowanie i importowanie ciasteczek dla poszczególnych domen lub globalnie. Idealne do badania sesji, czyszczenia śladów po audytach oraz ochrony przed przejęciem sesji (session hijacking).

---

### 4. Konfiguracja przeglądarki: Brak historii, brak ciasteczek i brak telemetrii

Aby upewnić się, że Firefox nie zapisuje żadnych danych lokalnie oraz nie wysyła raportów diagnostycznych do twórców (telemetria), przejdź do **Ustawienia** -> **Prywatność i bezpieczeństwo** (Privacy & Security) i skonfiguruj następujące opcje:

#### Historia i Ciasteczka:
*   W sekcji **Historia** (History), przy opcji *Program Firefox:* wybierz **będzie używał ustawień użytkownika** (Use custom settings for history).
*   Odznacz opcje:
    *   *Pamiętanie historii przeglądania i pobierania* (Remember browsing and download history).
    *   *Pamiętanie historii wyszukiwania i formularzy* (Remember search and form history).
*   Możesz również zaznaczyć opcję **Zawsze używaj trybu prywatnego** (Always use private browsing mode) – Firefox będzie wtedy działał w trybie, w którym po zamknięciu nie zostawia absolutnie żadnych śladów.
*   Zaznacz opcję **Wyczyszczenie historii przy zamykaniu programu Firefox** (Clear history when Firefox closes) i w ustawieniach obok zaznacz wszystko: ciasteczka, pamięć podręczną, aktywny stan zalogowania itp.
*   W sekcji **Ciasteczka i dane witryn** (Cookies and Site Data) zaznacz opcję **Usuwanie ciasteczek i danych witryn po zamknięciu programu Firefox** (Delete cookies and site data when Firefox is closed).

#### Blokowanie telemetrii (Wysyłanie danych do developera):
Zjedź niżej do sekcji **Kolekcja danych i korzystanie z nich przez program Firefox** (Firefox Data Collection and Use). **Odznacz** wszystkie poniższe opcje:
*   *Zezwolenie programowi Firefox na wysyłanie danych technicznych i o interakcji do firmy Mozilla* (Allow Firefox to send technical and interaction data to Mozilla).
*   *Zezwolenie programowi Firefox na instalowanie i przeprowadzanie badań* (Allow Firefox to install and run studies).
*   *Zezwolenie programowi Firefox na wysyłanie zgłoszeń awarii w imieniu użytkownika* (Allow Firefox to send backlogged crash reports on your behalf).

---

## Teoria Bezpieczeństwa: Dlaczego same proxy i VPN to za mało?

### 1. WebRTC i wyciek IP (WebRTC Leak)

Jak wspomniano wyżej, **WebRTC** służy do bezpośredniej komunikacji P2P. Standardowo, gdy przeglądarka chce połączyć się z serwerem, zapytanie przechodzi przez skonfigurowane proxy (np. Tor) lub tunel VPN. 

Jednak WebRTC działa inaczej. Aby nawiązać połączenie bezpośrednie z innym użytkownikiem (który może znajdować się za zapaporą sieciową NAT), WebRTC wysyła specjalne zapytania do serwerów **STUN** (Session Traversal Utilities for NAT). Te zapytania są wysyłane przez protokół **UDP** bezpośrednio z Twojego systemu operacyjnego. 

Ponieważ zapytania UDP w technologii WebRTC są wykonywane na poziomie niskopoziomowych gniazd sieciowych systemu, potrafią one odpytać system o adresy przypisane do wszystkich fizycznych kart sieciowych (np. Twoje lokalne IP w sieci domowej `192.168.1.X` oraz Twoje prawdziwe, publiczne IP od dostawcy internetu). Serwer STUN odsyła te dane z powrotem, a skrypt JavaScript osadzony na stronie internetowej może je odczytać. 

W ten sposób strona internetowa dowiaduje się, jaki jest Twój realny adres IP, mimo że ikona VPN na pasku zadań świeci się na zielono, a ruch HTTP idzie przez Tor.

---

### 2. DNS i jak można przez niego wpaść (DNS Leak)

**DNS (Domain Name System)** to system tłumaczący nazwy domen (np. `niebezpiecznik.pl`) na adresy IP serwerów (`185.117.84.120`).

#### Czym jest DNS Leak (Wyciek DNS)?
Kiedy łączysz się z siecią VPN, Twój ruch sieciowy jest szyfrowany. Jednak system operacyjny może nadal wysyłać zapytania o rozwiązanie nazw domen (DNS) do serwerów DNS Twojego lokalnego dostawcy internetu (ISP) zamiast przez bezpieczny tunel VPN. 

#### Jak przez to wpaść?
1.  **Dostawca internetu (ISP) widzi wszystko:** Twój ISP loguje każde zapytanie DNS wysłane z Twojego domowego IP. Nawet jeśli treść samej komunikacji ze stroną jest szyfrowana lub idzie przez VPN, Twój dostawca dokładnie wie, o jakie domeny pytałeś i w jakich godzinach.
2.  **Korelacja ruchu (Traffic Correlation):** Załóżmy, że wchodzisz na jakąś ukrytą stronę przez Tora lub VPN. Jeśli w tym samym momencie Twój system wyśle nieszyfrowane zapytanie DNS o tę domenę do serwera DNS Twojego ISP, obserwatorzy mogą łatwo powiązać to zapytanie (powiązane z Twoim prawdziwym nazwiskiem i adresem domowym) z ruchem, który chwilę później pojawił się na docelowym serwerze z adresu wyjściowego Tora/VPN.
3.  **Zatruwanie DNS (DNS Spoofing/Hijacking):** Korzystając z niezabezpieczonych serwerów DNS, jesteś narażony na to, że ktoś podstawi Ci fałszywy adres IP dla szukanej domeny, kierując Cię na podstawioną stronę phishingową (np. fałszywy panel logowania do banku).

---

### 3. Canvas Fingerprinting i potęga JavaScriptu

#### Czym są Canvasy?
Element `<canvas>` (płótno) w standardzie HTML5 służy do generowania grafiki w locie za pomocą skryptów JS. 

Gdy wchodzisz na stronę stosującą **Canvas Fingerprinting**, skrypt instruuje Twoją przeglądarkę, aby wyrenderowała ukryty (niewidoczny dla oka) obrazek zawierający tekst o określonym kroju pisma, cieniowanie, tekstury oraz efekty świetlne WebGL. 

Ponieważ renderowanie grafiki zależy od:
*   Modelu i producenta Twojej karty graficznej (GPU),
*   Wersji sterowników graficznych w systemie,
*   Zestawu zainstalowanych czcionek systemowych,
*   Silnika renderującego przeglądarki,
*   Wersji systemu operacyjnego,

ten sam obrazek zostanie wyrenderowany na poziomie pojedynczych pikseli w minimalnie inny sposób na różnych komputerach. Przeglądarka następnie konwertuje ten obraz na format Base64 i generuje z niego unikalny skrót (hash). Ten hash działa jak **cyfrowy odcisk palca**. Ponieważ prawdopodobieństwo, że dwie osoby mają identyczną konfigurację sprzętowo-programową jest minimalne, tracker może bezbłędnie identyfikować Cię na różnych stronach internetowych, nawet jeśli zmienisz IP, wyczyścisz ciasteczka i użyjesz trybu prywatnego.

#### Jak dużo informacji zostawiamy z włączonym JavaScriptem?
JavaScript pozwala stronom na odczytanie niemal pełnej specyfikacji Twojego komputera. Bez Twojej wiedzy strona może pobrać:
*   Liczbę rdzeni procesora (`navigator.hardwareConcurrency`),
*   Ilość pamięci RAM (w niektórych przeglądarkach),
*   Dokładny poziom naładowania baterii oraz informację, czy komputer jest podłączony do ładowania,
*   Pełną listę zainstalowanych wtyczek i czcionek,
*   Dokładną rozdzielczość ekranu, głębię kolorów i liczbę monitorów,
*   Dane z sensorów (np. akcelerometr w telefonie),
*   Dane behawioralne: precyzyjną dynamikę ruchów myszką i tempo pisania na klawiaturze (co pozwala na jednoznaczną identyfikację człowieka, a nie tylko maszyny).

#### Dlaczego całkowite wyłączenie JavaScriptu jest praktycznie niemożliwe?
Teoretycznie najprostszym sposobem na zachowanie anonimowości jest całkowite wyłączenie JS w przeglądarce. W praktyce jednak **współczesny internet bez JavaScriptu nie działa**.
1.  **Aplikacje SPA (Single Page Applications):** Większość nowoczesnych portali (np. Gmail, Twitter, panele bankowości, systemy rezerwacji) to aplikacje napisane w frameworkach takich jak React, Angular czy Vue. Serwer wysyła do przeglądarki pusty dokument HTML, który jest w całości budowany i renderowany dopiero przez JavaScript na Twoim komputerze. Bez JS zobaczysz tylko białą stronę.
2.  **Autoryzacja i Sesje:** Mechanizmy logowania, generowanie tokenów bezpieczeństwa (JWT), obsługa formularzy oraz zabezpieczenia przed botami (np. reCAPTCHA, Cloudflare) wymagają aktywnego JavaScriptu. Bez niego nie zalogujesz się na żadne konto.
3.  **Dynamiczna zawartość:** Bez JS nie uruchomią się odtwarzacze wideo, mapy interaktywne, czaty na żywo ani dynamicznie doładowywane komentarze.

Całkowite wyłączenie JS cofa nas do internetu z lat 90. Dlatego zamiast całkowicie go wyłączać, stosuje się podejście hybrydowe: **NoScript** do blokowania skryptów na podejrzanych stronach oraz narzędzia maskujące i szumujące (jak **CanvasBlocker** czy wbudowane mechanizmy ochrony przed śledzeniem w przeglądarce Tor Browser), które pozwalają na uruchamianie JS, ale karmią skrypty śledzące fałszywymi, znormalizowanymi danymi.

**Okej mam wsumie pare przykladow ktore wam to ladnie zobrazuja**

![[Zrzut ekranu_20260604_181325.png]]

> **Ta stronka ktora widzisz sluzy do testowania mojego autorskiego WAF jest to zwykly debuger ktory napisalem pod testy jednak skrypt posiada funkcje wykrywania czy uzytkownik ma obsluge JS, webrtc, canvasy etc.** 

> Tym razem na sporym farciku skrypt leaknal IP przez webrtc takie samo jak mam ustawione na VPN jednak jest to tez zaleta addonow do przegladarki i jej ustawien serio :) whatever 

  ![[pseudo-prawilna-hakerka/img/2.png]]


**Okej zajmijmy sie canvasami aby przejsc do tego panelu znajdz na pasku ikonke canvas blokera kliknij w nia i wybierz ustawienia, teraz dla przykladu zablokuje tworzenie canvasow... (nie jest to dobre rozwiazanie...) - raczej ustawiamy na fejkowanie to tylko przyklad**
  ![[pseudo-prawilna-hakerka/img/3.png]]

***Wuala fingerprint sie juz nie generuje***
  ![[4.png]]

**Dobra pora na obsluge JS wybieramy ikonke nojs i klikamy tu gdzie pokazalem aby zablokowac obsluge wszystkich skryptow jakie napotka przegladarka**
  ![[5.png]]
  
  ***Efekt? pieknie prawda a tak naprawde wiekszosc stron nie wpusci nas wcale :P*** 
  ![[6.png]]

**AHA! pragne zaznaczyc ze to co teraz zrobilismy z firefoxem nie daje nam gwarancji i pelnej anonimowosci... duzo pomoze ale to nie jest gwarancja bezpieczenstwa!** 

***Wypadalo by takze zajac sie systemem operacyjnym wgrac jakies fail2ban etc...***
- *dobra zajmijmy sie tym odrazu...*

### 1. Zapora i Ochrona: UFW + Fail2Ban

Na start musimy postawić mur na portach i ubić boty, które będą pukać do twojego systemu. Nawet jeśli masz pięknie skonfigurowane logowanie SSH po kluczach ED25519 (co jest jedyną słuszną drogą), chińskie i rosyjskie skanery będą ci non-stop zapychać logi próbując wbić się na roota.

Instalujemy pakiet:

Bash

```
sudo apt update && sudo apt install ufw fail2ban -y
```

**Konfiguracja UFW (Uncomplicated Firewall):** Zamykamy wszystko z zewnątrz, otwieramy to, co potrzebne. **WAŻNE:** Zanim odpalisz zaporę, upewnij się, że zezwoliłeś na port SSH, inaczej utniesz sobie dostęp do własnej maszyny!



```Bash
# Ustawiamy domyślne reguły: nic nie wchodzi, wszystko wychodzi
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Odblokowujemy port SSH (domyślnie 22, jeśli zmieniałeś - podaj swój)
sudo ufw allow ssh

# Uruchamiamy zaporę
sudo ufw enable
```

**Konfiguracja Fail2Ban:** Fail2ban skanuje logi systemowe i nakłada bany (poprzez iptables/ufw) na adresy IP, które zbyt wiele razy podały błędne hasło lub zachowują się podejrzanie.

1. Kopiujemy główny plik konfiguracyjny (nigdy nie edytujemy `jail.conf`, bo aktualizacja go nadpisze):
    

```Bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

2. Zjeżdżasz w dół do sekcji `[sshd]` i upewniasz się, że wygląda mniej więcej tak (zmieniasz `enabled` na `true`):
    

```Ini, TOML
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

_(Bantime = 3600 to godzina w izolatorze dla każdego bota po 3 błędnych próbach. Zapisz CTR+X / Y / Enter)._

3. Reset usługi:

```Bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

### 2. Uszczelnianie DNS (DNS over TLS via systemd-resolved)

To jest absolutnie kluczowe. Standardowy DNS lata po sieci **jawnym tekstem**. Twój dostawca internetu może nie wiedzieć co pobierasz, ale dokładnie widzi nazwy domen, które rozwiązuje twój system (np. `nmap.org`, `kali.org`, czy cele audytów).

Skonfigurujemy wbudowanego w Debiana demona `systemd-resolved`, aby cały ruch DNS z systemu szedł przez szyfrowany tunel TLS (DNS over TLS - DoT). Użyjemy do tego resolverów Quad9 (9.9.9.9 i 149.112.112.112) – to świetna, stawiająca na prywatność fundacja z siedzibą w Szwajcarii, a co najważniejsze, ich serwery w przeciwieństwie do popularnych rozwiązań komercyjnych nie ładują agresywnych blokad czy weryfikacji, które mogą uprzykrzać życie przy skanowaniu webówek.

1. Edytujemy plik resolvera:
    

```Bash
sudo nano /etc/systemd/resolved.conf
```

2. Odkomentowujesz (usuwasz `#`) i modyfikujesz linijki w sekcji `[Resolve]`, żeby wyglądały dokładnie tak:
    

```Ini, TOML
[Resolve]
DNS=9.9.9.9 149.112.112.112
#FallbackDNS=
Domains=~.
DNSSEC=yes
DNSOverTLS=yes
#MulticastDNS=yes
#LLMNR=yes
#Cache=yes
```

3. Zapisujesz i restartujesz usługę:
    

```Bash
sudo systemctl restart systemd-resolved
sudo systemctl enable systemd-resolved
```

4. **Weryfikacja:** Wpisz polecenie `resolvectl status`. Jeśli w sekcji twojego interfejsu (np. `eth0`) widzisz `DNSOverTLS: yes`, to jesteś szczelny. Twój DNS jest teraz szyfrowany na poziomie samego Debiana.
    

### 3. Auditd (Systemowy Konfident)

`Auditd` to potężne narzędzie, które pozwala ci śledzić każdą operację na poziomie jądra Linuxa. Kto, kiedy i czym dotknął danego pliku lub zmienił uprawnienia. Podczas testowania własnych, dziurawych apek do audytu, warto mieć kontrolę nad tym, co procesy wyprawiają w systemie plików.

1. Instalacja:
    

```Bash
sudo apt install auditd audispd-plugins -y
```

2. Konfiguracja reguł odbywa się w `/etc/audit/rules.d/audit.rules`. Możesz tam ustawić np. monitorowanie prób modyfikacji kluczowych plików:
    
```Bash
sudo nano /etc/audit/rules.d/audit.rules
```

Dopisz na dole, jeśli chcesz np. wiedzieć, kiedy jakikolwiek skrypt webowy (lub użytkownik) spróbuje dotknąć twoich plików haseł lub logów:



```Plaintext
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /var/log/auth.log -p wa -k auth_logs_accessed
```

_(flaga `-w` to ścieżka, `-p wa` to uprawnienia write/append, a `-k` to twój tag, po którym łatwo znajdziesz to w logach)._

3. Restart:

```Bash
sudo systemctl restart auditd
```

Jeśli chcesz sprawdzić, co się dzieje z twoim systemem pod kątem dodanych reguł, wpisujesz po prostu:


```Bash
sudo ausearch -k passwd_changes
```

### 4. Macchanger (Zmyłka sprzętowa)

Przydatne narzędzie z arsenału pentestera, jeśli atakujesz/audytujesz coś w sieci LAN lub łączysz się z obcego punktu dostępowego. Podmienia fizyczny adres MAC twojej karty sieciowej.


```Bash
sudo apt install macchanger -y
```

_(Podczas instalacji zapyta, czy ma zmieniać MAC automatycznie przy każdym połączeniu kabla/odpaleniu interfejsu. Polecam zaznaczyć **NIE**, żebyś robił to z pełną świadomością)._

Aby zmienić MAC na w pełni losowy na interfejsie np. `eth0` (najpierw musisz położyć interfejs):


```Bash
sudo ip link set dev eth0 down
sudo macchanger -r eth0
sudo ip link set dev eth0 up
```

_Uwaga: Jeśli robisz to na serwerze, do którego masz tylko zdalny dostęp (SSH), zignoruj macchangera, bo po położeniu interfejsu stracisz z nim kontakt!_

**Zdecydowanie dokupil bym do tego np mullvada (vpn) ktorego swoja droga bardzo latwo oplacic w pelnii anonimowo ;) za pomoca cryptowalut** 

https://mullvad.net/pl 

---

## Dodatek: Zaawansowane trasowanie ruchu – VPN over Tor vs Tor over VPN

Siemanko! Skoro w poprzedniej sekcji wspomniałem o **Mullvadzie**, czas wejść na wyższy poziom wtajemniczenia sieciowego. Często usłyszysz w świecie OPSEC pojęcia takie jak **VPN over Tor** oraz **Tor over VPN**. Choć brzmią podobnie, reprezentują zupełnie inne podejście do trasowania ruchu, mają skrajnie różne zastosowania, a ich konfiguracja (szczególnie tej pierwszej "sztuczki") wymaga żelaznej dyscypliny i odpowiednich reguł na firewallu (`iptables`/`ufw`).

Rozpiszmy to na czynniki pierwsze, bez owijania w bawełnę.

---

### 1. VPN over Tor (VPN przez sieć Tor) – Co to za "sztuczka"?

To jedna z najbardziej wykręconych konfiguracji sieciowych. Ruch z Twojego komputera płynie w następujący sposób:
`Twój system (Aplikacje) ➔ Tunel VPN ➔ Lokalny SOCKS5 (Tor) ➔ Sieć Tor (3 węzły) ➔ Serwer VPN ➔ Internet`

W praktyce oznacza to, że **zestawiasz tunel VPN wewnątrz sieci Tor**. Twój klient VPN łączy się z serwerem VPN nie bezpośrednio, ale przez lokalny port proxy SOCKS5 Tora (`127.0.0.1:9050`).

#### Na czym polega ta sztuczka i dlaczego jest genialna?
1. **Ominięcie blokad sieci Tor (Bypass Tor Bans):** To największa zaleta. Ogromna część internetu (Cloudflare, serwisy VOD, fora, sklepy) blokuje lub dręczy kapczami (CAPTCHA) użytkowników wchodzących z publicznych adresów IP węzłów wyjściowych Tora (Tor Exit Nodes). W tej konfiguracji docelowa strona widzi **IP serwera VPN**, a nie Tora! Przeglądasz internet anonimowo przez Tora, ale z bezproblemową prędkością i dostępem tradycyjnego VPN-a.
2. **Pełna anonimowość wobec dostawcy VPN:** Nawet jeśli Twój dostawca VPN loguje ruch lub zostanie przejęty przez służby, nie ma pojęcia, kim jesteś. Widzi jedynie, że połączenie do jego serwera przychodzi z losowego węzła wyjściowego sieci Tor. Jeśli dodatkowo założyłeś konto przez Tora i opłaciłeś je anonimowo (np. Monero lub gotówką), jesteś duchem.
3. **Szyfrowanie przed złośliwymi węzłami wyjściowymi Tora:** Ruch opuszczający sieć Tor i wchodzący do VPN jest w pełni zaszyfrowany kluczem VPN. Żaden złośliwy operator węzła wyjściowego Tora (Exit Node) nie podejrzy Twoich nieszyfrowanych pakietów ani nie wstrzyknie Ci złośliwego kodu.
4. **Ukrycie prawdziwego IP przed VPN i stronami:** Twoje IP jest chronione potrójną warstwą Tora, zanim w ogóle dotrze do serwera VPN.

#### Jakie są wady?
* **Brak dostępu do domen `.onion`:** Ponieważ cały ruch wychodzący z VPN-a trafia do czystego internetu (clearnet), tracisz możliwość bezpośredniego otwierania ukrytych usług `.onion`.
* **Prędkość żółwia:** Ruch przechodzi przez 3 szyfrowane węzły Tora rozrzucone po świecie, a potem przez serwer VPN. Opóźnienia (ping) będą gigantyczne (często ponad 1000 ms), a transfer mocno ograniczony.
* **Tylko protokół TCP:** Tor obsługuje wyłącznie ruch TCP. Oznacza to, że **nie możesz użyć protokołu WireGuard** (który działa tylko na UDP) bezpośrednio przez Tora. Musisz skonfigurować klienta VPN do pracy na protokole **OpenVPN w trybie TCP**.

---

#### Konfiguracja VPN over Tor za pomocą iptables (Debian/Ubuntu)

Aby ta sztuczka zadziałała i była w 100% bezpieczna, musimy postawić pancerny firewall. Jeśli klient VPN straci połączenie, system mógłby spróbować połączyć się bezpośrednio z internetem, co natychmiast ujawniłoby Twoje prawdziwe IP (wyciek/leak).

Nasza zapora musi realizować następujący plan:
1. Zezwolić usłudze Tor (użytkownik systemowy `debian-tor`) na swobodne łączenie się z internetem w celu budowania obwodów.
2. Zezwolić na ruch localhost (`lo`), aby klient VPN mógł połączyć się z lokalnym portem Tora (`127.0.0.1:9050`).
3. Zezwolić na pełen ruch przez wirtualny interfejs VPN (`tun0`).
4. **Zablokować cały pozostały ruch wychodzący** (żadne inne aplikacje nie mogą wysyłać pakietów bezpośrednio przez Twoją kartę sieciową `eth0` / `wlan0`).

Zapisz poniższy skrypt jako np. `/home/$USER/scripts_sh/vpn_over_tor_fw.sh`:

```bash
#!/bin/bash
# Skrypt konfigurujący twarde reguły iptables pod VPN over Tor

# Upewnij się, że skrypt działa jako root
if [ "$EUID" -ne 0 ]; then
  echo "[!] Uruchom ten skrypt z uprawnieniami sudo!"
  exit 1
fi

# Zmienne sieciowe
TOR_UID=$(id -u debian-tor)  # Pobranie UID użytkownika usługi Tor
VPN_INT="tun0"              # Interfejs wirtualny OpenVPN
LO_INT="lo"                 # Pętla zwrotna (localhost)

echo "[+] Czyszczenie starych reguł..."
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

echo "[+] Ustawianie domyślnej polityki (DROP)..."
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

echo "[+] Zezwalanie na ruch localhost (wymagany do komunikacji z proxy SOCKS5)..."
iptables -A INPUT -i $LO_INT -j ACCEPT
iptables -A OUTPUT -o $LO_INT -j ACCEPT

echo "[+] Zezwalanie usłudze Tor (UID: $TOR_UID) na bezpośrednie wyjście do sieci..."
iptables -A OUTPUT -m owner --uid-owner $TOR_UID -j ACCEPT

echo "[+] Zezwalanie na pełną komunikację przez interfejs VPN ($VPN_INT)..."
iptables -A INPUT -i $VPN_INT -j ACCEPT
iptables -A OUTPUT -o $VPN_INT -j ACCEPT

echo "[+] Zezwalanie na pakiety powiązane i nawiązane (ESTABLISHED, RELATED)..."
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

echo "[+] Reguły załadowane pomyślnie! Twój system jest teraz odcięty od bezpośredniego internetu."
echo "[+] Jedynym programem mogącym łączyć się bezpośrednio jest TOR."
```

Nadaj uprawnienia i uruchom skrypt:
```bash
chmod +x vpn_over_tor_fw.sh
sudo ./vpn_over_tor_fw.sh
```

#### Konfiguracja klienta OpenVPN pod Tor SOCKS5
Teraz musisz zmodyfikować plik konfiguracyjny `.ovpn` od swojego dostawcy VPN (np. pobrany plik konfiguracyjny OpenVPN TCP od Mullvada).

1. Otwórz plik `.ovpn` w edytorze tekstowym:
   ```bash
   nano mojalokalizacja_vpn.ovpn
   ```
2. Upewnij się, że konfiguracja wymusza protokół **TCP** (szukaj linijki `proto tcp` lub zmień `proto udp` na `proto tcp`).
3. Dopisz na samym dole następującą dyrektywę, która nakazuje klientowi OpenVPN tunelowanie połączenia przez lokalne proxy Tora:
   ```plain text
   socks-proxy 127.0.0.1 9050
   ```
4. Zapisz plik i uruchom połączenie VPN:
   ```bash
   sudo openvpn --config mojalokalizacja_vpn.ovpn
   ```
Gdy OpenVPN pomyślnie zestawi połączenie, Twój system będzie miał pełny dostęp do internetu, ale cały ruch będzie przechodził przez sieć Tor i wychodził przez serwer VPN!

---

### 2. Tor over VPN (Tor przez VPN) – Standardowa tarcza ochronna

To znacznie prostsza i powszechnie zalecana konfiguracja. Ruch płynie tak:
`Twój system ➔ Szyfrowany tunel VPN ➔ Serwer VPN (np. Mullvad) ➔ Sieć Tor ➔ Internet`

W tym scenariuszu najpierw łączysz się z VPN, a dopiero potem uruchamiasz Tora (np. Tor Browser).

#### Dlaczego to robimy? (Zalety)
1. **Ukrycie faktu korzystania z Tora przed ISP:** Twój dostawca internetu (oraz lokalne systemy monitorujące) widzą jedynie, że łączysz się z serwerem VPN. Nie wiedzą, że używasz Tora, co zapobiega automatycznemu oflagowaniu Twojego łącza domowego jako "podejrzane".
2. **Ochrona przed złośliwymi węzłami wejściowymi (Entry Nodes):** Pierwszy węzeł sieci Tor (Guard Node) nie widzi Twojego prawdziwego domowego adresu IP, tylko adres IP serwera VPN.
3. **Brak problemów z konfiguracją:** Działa "out-of-the-box" z dowolnym protokołem (w tym ultra-szybkim **WireGuard**).

#### Krok po kroku: Jak to zrobić na Mullvadzie?

Mullvad to idealny wybór do tej konfiguracji, ponieważ nie wymaga podawania żadnych danych przy rejestracji, posiada twardy, wbudowany Kill Switch i świetną aplikację dla Linuxa.

#### Krok 1: Instalacja oficjalnego klienta Mullvad VPN na Debianie 13
Dodajemy oficjalne repozytorium Mullvada i instalujemy aplikację:

```bash
# Pobranie klucza GPG
sudo curl -fsSLo /usr/share/keyrings/mullvad-keyring.asc https://repository.mullvad.net/deb/mullvad-keyring.asc

# Dodanie repozytorium do źródeł apt
echo "deb [signed-by=/usr/share/keyrings/mullvad-keyring.asc] https://repository.mullvad.net/deb/stable main" | sudo tee /etc/apt/sources.list.d/mullvad.list

# Instalacja aplikacji
sudo apt update && sudo apt install mullvad-vpn -y
```

#### Krok 2: Konfiguracja Mullvada (CLI lub GUI)
Możesz zarządzać Mullvadem bezpośrednio z terminala:

```bash
# Zalogowanie się na swoje konto (podaj swój 16-cyfrowy numer)
mullvad account login TWÓJ_NUMER_KONTA

# Włączenie lokalnego Kill Switcha (blokowanie ruchu poza VPN)
mullvad lockdown-mode set on

# Wybór protokołu na WireGuard (najszybszy i najbezpieczniejszy)
mullvad relay set tunnel-protocol wireguard

# Połączenie z losowym, bezpiecznym serwerem (np. Szwajcaria lub Szwecja)
mullvad connect
```

Możesz też sprawdzić status połączenia:
```bash
mullvad status
```

#### Krok 3: Odpalenie sieci Tor
Gdy Mullvad jest połączony i chroni cały Twój system, po prostu uruchamiasz Tora:

* **Opcja A (Tor Browser - zalecana do przeglądania stron):** Pobierz i uruchom oficjalną przeglądarkę Tor Browser. Automatycznie połączy się ona z siecią Tor przez bezpieczny, zaszyfrowany tunel Mullvada.
* **Opcja B (Systemowy Tor + Proxychains do narzędzi pentestu):** Uruchom usługę Tor w tle:
  ```bash
  sudo systemctl start tor
  ```
  Teraz możesz odpalać dowolne narzędzia (np. nmap czy sqlmap) przez sieć Tor, wpisując przed komendą `proxychains4`:
  ```bash
  proxychains4 nmap -sT -PN target_ip
  ```
  Pakiety najpierw przejdą przez serwer Mullvad, potem przez 3 węzły Tora, a na końcu uderzą w cel!

---

### 💡 Alternatywa: Mullvad Browser – Prywatność Tora z prędkością VPN-a

Jeśli zależy Ci na pancernej ochronie przed profilowaniem (Canvas Fingerprinting, blokada WebRTC, blokada skryptów itp.), ale nie chcesz cierpieć katuszy związanych z niską prędkością sieci Tor, Mullvad we współpracy z **Tor Project** stworzył dedykowaną przeglądarkę – **Mullvad Browser**.

Jest to zmodyfikowana wersja Tor Browser, która:
* Posiada identyczne mechanizmy ochrony przed identyfikacją przeglądarki (fingerprinting).
* **Nie korzysta z sieci Tor** – zamiast tego wysyła ruch bezpośrednio przez Twoje połączenie sieciowe (czyli przez Mullvad VPN).
* Daje Ci maksymalną prędkość Twojego łącza przy zachowaniu najwyższych standardów prywatności przeglądarki.

Przeglądarkę Mullvad Browser możesz pobrać bezpośrednio ze strony Mullvada lub za pomocą menedżera pakietów, jeśli jest dostępny w Twojej dystrybucji. To rewelacyjny kompromis do codziennej, bezpiecznej pracy!

--------------------------------------

## Myslicie ze nie zejde na bardziej paranoiczny LVL? B-) 

**A co jak by do aktualnej konfiguracji dolazyc jeszcze virtualboxa z systemem whonix ktory ma swoja bramke Tor'a?**

> *ogolnie jest to jeden z gorszych pomyslow chyba ze pojdziemy na pewne ustepstwa, czemu jest to chujowy pomysl? ze wzgledu na czas jaki bedzie nam zajmowal jaki kolwiek skan... omuij boze komenda **nmap -v -A < ip albo url >** trwala by przy dobrych wiatrach pare dni :P*  

## **Ale jesli jednak ktos by chcial sprobowac...**


### 1. Architektura Sieciowa: Jak będzie wyglądał twój ślad?

Kiedy odpalisz ten zestaw, pakiety wychodzące z twojej przeglądarki w Whonix Workstation przejdą przez absolutne piekło kryptograficzne, zanim dotrą do serwera docelowego. Wygląda to tak:

**Etap 1 (Twój Host - Debian 13):** Twój system -> Twój ISP -> Sieć Tor Hosta (Entry -> Middle -> Exit) -> Serwer VPN

**Etap 2 (Maszyna Wirtualna - Whonix):** Z punktu widzenia Whonixa, jego "dostawcą internetu" staje się twój serwer VPN. Whonix bierze ten zaszyfrowany ruch i pakuje go w _swoją własną_ sieć Tor. Serwer VPN -> Whonix Tor Entry -> Whonix Tor Middle -> Whonix Tor Exit -> SERWER DOCELOWY.

**Wynik:** Twój ruch przelatuje przez minimum **7 skoków** w różnych krajach, z czego 6 to węzły Tora, a w środku siedzi tunel VPN. Twój "Entry Node" dla głównego, widocznego Tora to adres IP komercyjnego VPN-a, a nie twój domowy internet.

### 2. Dlaczego to jest genialne? (Zalety)

- **Ostateczna Izolacja (Compartmentalization):** Jeśli złośliwy kod na audytowanej stronie zdoła wyjść z przeglądarki (tzw. sandbox escape) i zainfekuje Whonix Workstation, nie dowie się absolutnie niczego. Whonix Workstation ma tylko lokalne IP (zwykle 10.152.152.x) i nie ma fizycznego dostępu do sprzętu.
    
- **Ochrona przed deanonimizacją Entry Guard:** Ataki na sieć Tor często polegają na kontrolowaniu węzła wejściowego i wyjściowego. W tym układzie, nawet jeśli ktoś skompromituje twojego Whonixowego Tora, dotrze tylko do adresu IP twojego VPN-a. Żeby pójść dalej, musiałby złamać logi VPN-a, a potem jeszcze rozplątać pierwszą pętlę Tora na twoim hoście. Powodzenia.
    

### 3. Brutalna Rzeczywistość (Wady i "Taki chuj")

Zanim zaczniesz się cieszyć, musimy pogadać o fizyce i protokołach. Zbudowałeś właśnie potwora zwanego **Tor-over-Tor** (przedzielonego tylko VPN-em). Projekt Tor oficjalnie odradza takie praktyki z dwóch powodów:

- **TCP Meltdown (Załamanie protokołu TCP):** Tor przesyła dane paczkami, używając protokołu TCP, który gwarantuje dostarczenie pakietu. Kiedy pakujesz TCP wewnątrz OpenVPN-a (który wymusiliśmy na TCP), a to wszystko wewnątrz drugiego Tora, powstaje chaos. Jeśli jeden pakiet na poziomie Whonixa się spóźni, wewnętrzny TCP prosi o retransmisję. Ale zewnętrzny TCP (z hosta) też widzi opóźnienie i też retransmituje. Sieć zaczyna dusić się własnymi duplikatami.
    
- **Prędkość i Timeouty:** Twój ping będzie liczony w sekundach (często 2000-5000 ms). Przepustowość spadnie do kilkunastu kilobajtów na sekundę.
    

**Co to oznacza dla pentestera?** Zapomnij o używaniu zautomatyzowanych skanerów jak FFUF, Dirb, czy Dirsearch z wnętrza Whonixa. Wszystkie zapytania HTTP będą łapać `timeout` (upłynięcie czasu żądania). Skaner uzna, że strona nie istnieje, a w rzeczywistości to twój 7-warstwowy tunel nie zdążył odpowiedzieć. Ten setup nadaje się wyłącznie do bardzo powolnego, ręcznego rekonesansu (czytania forum, analizy kodu źródłowego w przeglądarce).

### 4. Jak to skonfigurować (Setup dla debila)

Jeśli mimo ostrzeżeń chcesz to zrobić i poczuć się jak Edward Snowden na sterydach, oto jak to zepniemy:

**Krok 1: Przygotowanie Hosta (Debian 13)** Musisz mieć odpalony nasz skrypt `vpn_over_tor_firewall.sh` z poprzedniej lekcji. Sprawdź komendą `curl ifconfig.me` czy twój adres IP to adres serwera VPN. Wszystko musi wychodzić przez interfejs `tun0`.

**Krok 2: Instalacja VirtualBoxa**

Bash

```
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack -y
```

**Krok 3: Import Whonixa**

1. Wchodzisz (przez opancerzonego liska) na stronę Whonix.org i pobierasz obraz dla VirtualBoxa (plik `.ova` - zawiera Gateway i Workstation).
    
2. Otwierasz VirtualBoxa -> `Plik` -> `Importuj urządzenie programowe` -> Wybierasz pobrany plik.
    
3. Ważne: **Nic nie zmieniaj w ustawieniach sieciowych maszyn!** Whonix automatycznie konfiguruje swoje interfejsy. Gateway używa trybu NAT (czyli bierze internet od twojego hosta), a Workstation używa "Sieci wewnętrznej" (podłącza się do Gatewaya).
    

**Krok 4: Magia routingu (Dzieje się sama)** Z racji, że twój host (Debian 13) jest spięty naszymi twardymi regułami IPTables, VirtualBox chcąc dać internet dla Whonix Gateway, zostanie bezwzględnie przekierowany do interfejsu `tun0`. Nie musisz pisać dodatkowych reguł. VirtualBox jest ślepo posłuszny tablicy routingu twojego Debiana.

**Krok 5: Odpalenie systemów**

1. Zawsze najpierw uruchamiasz w VirtualBoxie maszynę **Whonix Gateway**. Czekasz aż się załaduje i połączy z siecią Tor (będzie używał do tego twojego VPN-a). Puszczasz tam komendę `upgrade-nonroot` w terminalu, żeby się zaktualizował.
    
2. Dopiero gdy Gateway działa, odpalasz **Whonix Workstation**.
    

### Podsumowanie

Zamieszałeś w głowie systemom śledzącym tak bardzo, że NSA pewnie by musiało zużyć budżet średniego państwa na prąd, żeby cię wyśledzić. Masz sprzętową zmyłkę (MAC), ochronę systemu (AppArmor w Whonixie), uszczelniony DNS hosta, Tora wpiętego w VPN-a, który karmi drugiego Tora.

Do uderzeń ukierunkowanych i bezpiecznego przeglądania Darknetu (np. żeby sprawdzić, czy dane twojego klienta tam nie wyciekły)? Genialne. Do agresywnego skanowania podatności i Fuzzingu? Bezużyteczne z powodu lagów.

Decyzja należy do Ciebie, architektura stoi gotowa! Co bierzemy na warsztat w następnej kolejności?


### Jesli w twojej glowie zrodzilo sie pytanie 

>
>***a w jaki sposob mogl bym sprawic zeby bylo to optymalne i mialo sens?***
>


**Zadałeś bardzo mądre pytanie.** Skoro ta 7-warstwowa kurwa (Debian -> VPN -> Tor -> Whonix -> Tor) dusi się własnymi pakietami, to jak to zrobić, żeby "zjeść ciastko i mieć ciastko"? Czyli zachować absolutną paranoję i anonimowość, ale jednocześnie móc odpalić Nuclei czy FFUF na pełnej kurwie, nie czekając tygodnia na wynik?

Odpowiedź brzmi: **Praw fizyki i architektury TCP nie oszukasz. Musisz zmienić podejście i rozdzielić warstwę anonimizacji od warstwy uderzeniowej.**

Zamiast pchać ciężki ruch (skanowanie 10,000 portów lub tysięcy ścieżek webowych) przez sieć Tor ze swojego domowego komputera, profesjonalni pentesterzy (i ci z tej ciemniejszej strony mocy) budują architekturę opartą o **Jump Server (Serwer Przesiadkowy / VPS)**.

Oto jak zrobisz to optymalnie i z sensem.

### Nowa Architektura: "The Ghost VPS"

Zamiast robić "Incepcję" lokalnie na swoim biurku, przenosimy ciężki sprzęt do chmury, a Twój zabetonowany Debian służy tylko jako terminal dowodzenia.

**Jak to wygląda w praktyce?**

1. **Twój komputer:** Twój Debian 13 z odpalonym VPN-over-Tor (nasz skrypt iptables z poprzedniej lekcji) lub po prostu Whonix. Jesteś całkowicie ukryty.
    
2. **Kupno "Brudnego" VPS'a:** Przez przeglądarkę w Torze kupujesz tani serwer VPS (Virtual Private Server) u dostawcy offshore (np. w Szwajcarii, Islandii czy Rumunii), który akceptuje płatności w kryptowalutach (najlepiej Monero - XMR, bo jest nie do wyśledzenia). Podajesz fałszywe dane, zero powiązania z Twoją osobą.
    
3. **Instalacja arsenału:** Logujesz się na ten serwer po SSH i tam wgrywasz nasz skrypt `install_tools.sh` z pierwszego posta. To ten VPS ma teraz Nmapa, Nuclei i FFUF.
    
4. **Atak / Audyt:** Ze swojego w 100% anonimowego Debiana w domu, łączysz się po SSH przez Tora z Twoim nowym VPS-em. Wydajesz komendę. Skanowanie leci z poziomu VPS-a na serwer docelowy.
    

### Dlaczego to ma potężny sens i rozwiązuje wszystkie problemy?

- **Prędkość i brak timeoutów (Brak laga):** Skanery odpalone na VPS-ie korzystają z jego bezpośredniego, potężnego łącza do internetu (często 1 Gbps lub 10 Gbps). Skan idzie pełną prędkością, bez żadnych opóźnień.
    
- **Omijanie Cloudflare i banów na Exit Nodes:** Serwer docelowy (audytowana aplikacja) nie widzi ruchu z sieci Tor! Widzi czyste, normalne IP komercyjnego dostawcy chmurowego. Żadne WAF-y (Web Application Firewalls) ani CAPTCHE nie będą Ci blokować skanerów z automatu.
    
- **Pełna obsługa UDP:** Z racji, że to VPS uderza w cel bezpośrednio, możesz używać skanowania UDP w Nmapie do woli.
    
- **Ostateczna anonimowość:** Twój ruch od Ciebie do VPS-a to tylko czysty tekst w terminalu SSH (zużywa bajty danych, więc Tor na luzie to dźwiga bez laga). Nawet jeśli cel zorientuje się, że jest audytowany i zgłosi nadużycie (Abuse), zablokują lub wyłączą VPS-a. Twój prawdziwy adres domowy (ani IP Twojego prywatnego VPN-a) nigdy nie dotknął serwera docelowego. Spalasz VPS-a za 5 dolarów, kupujesz następnego.
    

### Co jeśli jednak MUSISZ skanować ze swojego lokalnego komputera (Whonixa)?

Jeśli z jakiegoś powodu nie chcesz stawiać zewnętrznego VPS-a i upierasz się przy skanowaniu z wnętrza Whonixa, musisz dostosować narzędzia do faktu, że masz gigantycznego pinga (laga). Standardowo skanery wysyłają setki zapytań na sekundę i czekają krótko na odpowiedź. W Torze to skończy się rzezią i fałszywymi negatywami (False Negatives).

Musisz "wykastrować" narzędzia, żeby działały wolno, ale skutecznie:

> **1. FFUF (Fuzzing katalogów w webówkach)** Zapomnij o wielowątkowości. Musisz drastycznie zwiększyć czas oczekiwania na odpowiedź i spowolnić rate-limit (ilość żądań). `ffuf -w wordlist.txt -u https://cel.com/FUZZ -t 1 -timeout 30 -rate 2` _(Flaga `-t 1` to tylko 1 wątek, `-timeout 30` każe mu czekać aż pół minuty na odpowiedź, `-rate 2` wysyła tylko 2 żądania na sekundę)._
> 
> **2. Nmap (Skanowanie portów)** Musisz nakazać Nmapowi traktować połączenie jak bardzo, bardzo kiepskie: `nmap -sT -Pn -n --max-retries 4 --scan-delay 2s --max-scan-delay 10s -T2 cel.com` _(Zmuszasz go do pełnego TCP Connect `-sT`, wyłączasz pingowanie `-Pn`, każesz robić 2 sekundy przerwy między pakietami `--scan-delay 2s` i używasz powolnego szablonu `-T2`)._

> **3. Nuclei (Skanowanie podatności)** Podobnie, musisz zmniejszyć równoległość i zwiększyć czas oczekiwania: `nuclei -u https://cel.com -c 2 -bs 2 -timeout 25 -retries 3` _(Tylko dwa jednoczesne zapytania `-c 2`, małe paczki `-bs 2` i gigantyczny timeout)._

### Mysle ze wiekszosc osob wlasnie uswiadomila sobie z jakim typem czlowieka ma tu do czynienia hah

***oczywiscie ze potafil bym to pokomplikowac jeszcze bardziej XD*** 

**Oraz mam nadzieje ze wiedza ktora przekazuje a nie powinienem :) takze zostanie doceniona jesli chcecie nie obraze sie za malego tipka w XMR (jesli nie wiesz co to XMR) - nie wiem czemu to czytasz :'P** 

-------------------------------------------------------
#### ☕ Tip Me / Wyszło przydatne? Postaw piwo!

Jeśli moje narzędzia lub poradniki okazały się dla Ciebie przydatne i chcesz wesprzeć mój rozwój, możesz sypnąć drobnymi na poniższe adresy. Szanujemy prywatność, więc Monero zawsze mile widziane! 🥷

| Kryptowaluta | Sieć (Network) | Adres Portfela |
| :--- | :--- | :--- |
| 🟠 **Bitcoin (BTC)** | Bitcoin | `bc1q_TWOJ_ADRES_BTC_TUTAJ` |
| 🦇 **Monero (XMR)** | Monero | `4_TWOJ_ADRES_XMR_TUTAJ` |
| 💎 **Ethereum (ETH)** | ERC-20 | `0x_TWOJ_ADRES_ETH_TUTAJ` |
| 💵 **Tether (USDT)** | TRC-20 (Tron) | `T_TWOJ_ADRES_USDT_TUTAJ` |

-------------------------------------------------------------------

**Ale dobra wracajac do tematu pamietaj ze te rozpisane konfiguracje mozesz uzywac naprzemiennie, nie musisz ciagle siedzec na tej kurwie 3000 warstwowej i sie meczyc xD**

> 	 *co jak co ten przyklad z konca to chyba nada sie w przypadku jak zechcesz*  *poleciec do korei polnocnej i zostac anty-rzadowym bialym dziennikarzem

## Dobra przejdzmy do czegos naprawde prostego...

**Przygotowałem dla Ciebie kompletną paczkę (plik ZIP u góry) składającą się z trzech plików. Oczywiście zadbałem o to, aby nasza aplikacja nie korzystała z żadnych zewnętrznych bibliotek (CDN), więc wszystko zadziała w pełni lokalnie i błyskawicznie, bez stresu, że jakiś zewnętrzny WAF czy Cloudflare zablokuje Ci arkusze stylów.**

### Co znajdziesz w paczce?

Paczka zawiera trzy pliki, które ze sobą współpracują:

1. **`setup_lamp.sh`** – Główny skrypt dla Twojego Debiana 13. Instaluje i konfiguruje stertę LAMP (Apache2, MariaDB, PHP), tworzy nową bazę danych i podmienia domyślną stronę Apache na naszą podatną apkę.
    
2. **`database.sql`** – Struktura bazy danych, która tworzy tabelę użytkowników i wstrzykuje do niej "tajne" flagi (notatki), które będziesz musiał wykraść za pomocą SQLi.
    
3. **`index.php`** – Nasz właściwy cel ataku. Prosta strona internetowa napisana w czystym PHP bez żadnego filtrowania wejścia.
    

### 1. Skrypt instalacyjny (`setup_lamp.sh`)

Ten skrypt odwala całą brudną robotę. Instaluje serwer, tworzy użytkownika bazy danych, importuje tabele i nadaje uprawnienia dla plików przeglądarki.



```Bash
#!/bin/bash
# Skrypt automatycznej instalacji i konfiguracji srodowiska LAMP na Debian 13

set -e
echo "[+] Aktualizacja pakietow..."
sudo apt update

echo "[+] Instalacja Apache2, MariaDB (MySQL) oraz PHP..."
sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-mysql

echo "[+] Konfiguracja i uruchamianie uslug..."
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl enable mariadb
sudo systemctl start mariadb

echo "[+] Tworzenie bazy danych oraz uzytkownika laboratoryjnego..."
sudo mysql -e "CREATE DATABASE IF NOT EXISTS lab_db;"
sudo mysql -e "CREATE USER IF NOT EXISTS 'lab_user'@'localhost' IDENTIFIED BY 'SilneHasloBazy123!';"
sudo mysql -e "GRANT ALL PRIVILEGES ON lab_db.* TO 'lab_user'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

echo "[+] Importowanie podatnej struktury bazy danych..."
if [ -f "./database.sql" ]; then
    sudo mysql lab_db < ./database.sql
fi

echo "[+] Wdrażanie podatnej aplikacji webowej (index.php)..."
if [ -f "./index.php" ]; then
    sudo rm -f /var/www/html/index.html
    sudo cp ./index.php /var/www/html/index.php
    sudo chown -R www-data:www-data /var/www/html/
    sudo chmod -R 755 /var/www/html/
fi

echo "[+] Restartowanie serwera Apache..."
sudo systemctl restart apache2
echo "Twoje lokalne laboratorium jest gotowe: http://localhost/"
```

### 2. Dziurawy Kod (Co będziemy atakować?)

Zajrzyjmy w serce problemu, czyli co dokładnie zepsułem w pliku `index.php`, żebyś mógł to uderzyć.

**Podatność SQL Injection (SQLi):** Klasyk gatunku. Parametr `id` jest przekazywany w żądaniu GET i dosłownie doklejany surowym tekstem do zapytania SQL, zamiast użycia tzw. _Prepared Statements_. Dodatkowo zostawiłem włączone wyświetlanie błędów MySQL, więc będziesz mógł przećwiczyć techniki _Error-Based SQL Injection_.



```PHP
// Fragment podatnego kodu index.php
if (isset($_GET['id'])) {
    $id = $_GET['id'];
    
    // PODATNOSC: Bezposrednie wstrzykniecie zmiennej do zapytania SQL
    $query = "SELECT id, username, role, secret_note FROM users WHERE id = " . $id;
    $result = $conn->query($query);
}
```

**Podatność Cross-Site Scripting (XSS):** Zrobiłem mały symulator sekcji "dodaj komentarz". Formularz leci metodą POST, a PHP odbiera go i wypluwa bezpośrednio w strukturę HTML. Żadnego weryfikowania typu `htmlspecialchars()` czy `strip_tags()`. Zwykły, bezczelny `echo`.



```PHP
// Fragment podatnego kodu index.php
$xss_output = "";
if (isset($_POST['comment'])) {
    // PODATNOSC: Brak filtrowania znakow specjalnych HTML/JS
    $xss_output = $_POST['comment'];
}
```

W samej strukturze strony znajduje się wtedy: `<div> <?php echo $xss_output; ?> </div>`, co oznacza, że jeśli wrzucisz tam tag `<script>`, przeglądarka odczyta to jako pełnoprawny kod JavaScript w źródle strony.

### Jak to uruchomić panie monter?

1. Pobierz plik ZIP udostępniony na samej górze.
    
2. Wypakuj wszystko do jednego folderu na swoim Debianie (np. `~/Dokumenty/lab_pentest`).
    
3. Wejdź w terminalu do tego folderu:
    
    
    
    ```Bash
    cd ~/Dokumenty/lab_pentest
    ```
    
4. Nadaj uprawnienia do wykonania na skrypt instalacyjny:
    
    
    
    ```Bash
    chmod +x setup_lamp.sh
    ```
    
5. Odpal środowisko:
    
    
    
    ```Bash
    ./setup_lamp.sh
    ```
    

Gdy skrypt skończy mielić, odpalasz zbrojnego Firefoxa i wpisujesz w pasek adresu: `http://localhost/` lub swój lokalny adres maszyny, na której to postawiłeś.

Masz teraz własny, prywatny poligon testowy, na którym całkowicie legalnie przetestujesz komendy z palca na obcinanie zapytań SQL i wykonywanie szkodliwych payloadów JavaScript. Daj znać, jak już postawisz maszynę – w następnym kroku pokażę Ci, jak podejść do tego ręcznie, bez używania zautomatyzowanych kombajnów. Gotowy na włamanie do własnego serwera? B-)

**tu filmik** 

### 1. WhatWeb (Rozpoznanie / Fingerprinting)

**Do czego służy?** WhatWeb to tzw. _Web Scanner_ nowej generacji, ale ja wolę go nazywać "cyfrowym detektywem". Nie służy do łamania czy hakowania. Służy do **fingerprintingu**, czyli identyfikacji technologii, na których stoi strona. Zanim uderzysz w cel, musisz wiedzieć, z czym masz do czynienia. WhatWeb po jednym strzale powie Ci: jaki to CMS (WordPress, Joomla?), jaka wersja PHP, jaki serwer (Apache/Nginx/IIS), jakich bibliotek JS używa strona (jQuery, React), a nawet czy wykrył jakieś firewalle aplikacyjne (WAF).

**Poziomy Agresji (Aggression Levels):** To kluczowa funkcja WhatWeba. Definiuje, jak bardzo hałasujesz w logach serwera.

- **-a 1 (Stealthy / Pasywny):** Domyślny. Robi tylko jedno żądanie HTTP GET. Cichy jak duch.
    
- **-a 3 (Aggressive):** Uderza głębiej. Zgaduje foldery, pobiera więcej plików, szuka ukrytych ścieżek. Głośniejszy, ale wykrywa więcej.
    
- **-a 4 (Heavy):** Zrobi w logach admina jesień średniowiecza. Skanuje absolutnie każdy plugin i każdy możliwy plik konfiguracyjny.
    

**Najważniejsze parametry i przykłady użycia:**


```Bash
# Szybki, cichy strzał w pojedynczy cel (poziom 1)
whatweb example.com

# Pełny, gadatliwy skan (Verbose) - pokaże Ci dokładnie, dlaczego wykrył dany soft
whatweb -v example.com

# Agresywny skan szukający ukrytych technologii
whatweb -a 3 https://example.com

# Skanowanie masowe z pliku (np. lista subdomen)
whatweb -i lista_subdomen.txt

# Podmiana User-Agenta (żeby nie przedstawiać się jako skaner)
whatweb --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)" example.com

# Zapisanie wyników do ładnego pliku JSON (idealne do dalszej obróbki skryptami)
whatweb --log-json=wyniki.json example.com
```

### 2. Nikto (Stary wyga od brudnej roboty)

**Do czego służy?** Nikto to napisany w Perlu klasyk, który ma już swoje lata, ale wciąż potrafi wykopać trupa z szafy. To typowy **skaner serwerów WWW**. Nie jest subtelny. Napierdala w cel tysiącami zapytań (często około 6500+ testów), szukając: starych, dziurawych wersji serwerów, domyślnych plików konfiguracyjnych (których admin zapomniał usunąć), niezabezpieczonych katalogów (np. `.git/`, `backup/`) i podstawowych problemów z nagłówkami (np. brak nagłówków chroniących przed Clickjackingiem).

Jeśli admin zostawił na serwerze plik `phpinfo.php` albo starą paczkę `baza.zip`, Nikto to znajdzie.

**Najważniejsze parametry i przykłady użycia:**


```Bash
# Standardowy skan pojedynczego celu
nikto -h http://10.10.10.10

# Skanowanie konkretnego portu (jeśli webówka nie stoi na 80/443)
nikto -h 10.10.10.10 -p 8080

# Skanowanie z omijaniem prostych systemów IDS/WAF (Techniki Evasion)
nikto -h example.com -evasion 1   # 1 - Dodaje losowe znaki kodowania URL (np. %2e zamiast kropki)
nikto -h example.com -evasion A   # A - Używa "powrotu karetki" w żądaniach

# Parametr -Tuning (Wybieranie konkretnych testów, żeby skrócić czas)
# 1 - Pliki konfiguracyjne, 4 - XSS, 8 - Wykonywanie komend w systemie (RCE), 9 - SQLi
nikto -h example.com -Tuning 1489

# Wymuszenie skanu przez nasze proxy Tor (127.0.0.1:9050) - jeśli nie używasz naszego skryptu iptables
nikto -h example.com -useproxy http://127.0.0.1:9050

# Zapisanie wyników do formatu HTML (generuje fajny, czytelny raport)
nikto -h example.com -Format htm -o raport_nikto.html
```

### 3. Nuclei (Nowoczesna machina zagłady)

**Do czego służy?** Nuclei (od ProjectDiscovery) to obecnie absolutny król audytów webowych. W przeciwieństwie do Nikto, Nuclei nie ma zahardcodowanych testów. Działa w oparciu o **szablony YAML (Templates)**. Baza szablonów jest aktualizowana codziennie przez społeczność z całego świata. Kiedy wychodzi nowa, krytyczna podatność (np. CVE na serwery Exchange, Log4j, czy luki w pluginach WordPressa), po kilku godzinach w bazie Nuclei ląduje szablon, a Ty od razu możesz przeskanować pod tym kątem setki swoich maszyn.

Jest piekielnie szybki (pisany w Go), potrafi szukać wycieków danych, podatności typu RCE, XSS, przejęć subdomen i miskonfiguracji w chmurze (AWS/Azure).

**Najważniejsze parametry i przykłady użycia:**

Zanim w ogóle go odpalisz, po instalacji zawsze wpisz `nuclei -ut` (Update Templates), żeby pobrać najnowsze wektory ataków!


```Bash
# Prosty strzał we wszystkie podstawowe podatności dla jednego adresu
nuclei -u https://example.com

# Złoto: Skanowanie całej listy adresów URL z pliku
nuclei -l moje_cele.txt

# Skanowanie z użyciem konkretnych tagów (np. sprawdzamy TYLKO podatności cve i luki w wordpress)
nuclei -u https://example.com -tags cve,wordpress

# Skanowanie konkretnym rodzajem szablonu (np. tylko szukanie ujawnionych tokenów i haseł)
nuclei -u https://example.com -t exposures/

# Opcje optymalizacji i agresywności (szybkość skanowania)
# -c (concurrency) - Ilość szablonów odpalanych naraz
# -rl (rate-limit) - Ilość żądań na sekundę
nuclei -l cele.txt -c 50 -rl 150

# Włączenie trybu automatycznego (Auto-Scan)
# Technologia Wappalyzer połączy się z rozpoznaniem z WhatWeb i odpali tylko te szablony, 
# które pasują do wykrytych technologii (oszczędza to mnóstwo czasu!)
nuclei -u https://example.com -as

# Skanowanie z wypuszczaniem ruchu przez naszego Tora (Proxy)
nuclei -u https://example.com -proxy socks5://127.0.0.1:9050

# Zapis wyników do pliku markdown
nuclei -u https://example.com -o wyniki_nuclei.md
```


### 4. Nmap (Wszechwidzące oko sieci)

**Do czego służy?** Nmap (Network Mapper) to absolutny ojciec chrzestny skanowania infrastruktury. O ile WhatWeb czy Nuclei uderzają w aplikacje webowe (warstwa 7 OSI), o tyle Nmap sprawdza, co w ogóle jest otwarte na serwerze (warstwa 3 i 4). Zanim zaczniesz grzebać w stronach, musisz wiedzieć, czy cel nie zostawił otwartego portu SSH, bazy danych MySQL, czy panelu FTP z dostępem dla gościa. Nmap mapuje sieć, wykrywa wersje usług, system operacyjny, a dzięki silnikowi NSE (Nmap Scripting Engine) potrafi samodzielnie wyłapywać (a nawet eksploitować) podstawowe podatności.

**Poziomy Szybkości (Timing Templates):** Od `-T0` (Paranoik – ekstremalnie wolny, omija IDS) do `-T5` (Insane – napierdala jak z karabinu maszynowego, głośny jak diabli). Najczęstszy złoty środek to `-T4`.

**Najważniejsze parametry i przykłady użycia:**



```Bash
# Prosty, standardowy skan 1000 najpopularniejszych portów
nmap 10.10.10.10

# Skanowanie agresywne: Wykrywa system operacyjny (-O), wersje usług (-sV), odpala domyślne skrypty (-sC)
nmap -A -T4 10.10.10.10

# Cichy skan (Stealth SYN scan) - nie nawiązuje pełnego połączenia TCP, trudniejszy do wykrycia przez zapory
nmap -sS 10.10.10.10

# Skanowanie WSZYSTKICH 65535 portów (zajmuje trochę czasu, ale nic nie ukryje)
nmap -p- 10.10.10.10

# Ultra szybki skan (wymusza wysyłanie minimum 1000 pakietów na sekundę)
nmap -p- --min-rate 1000 10.10.10.10

# Wykorzystanie skryptów NSE: Szukanie konkretnych podatności (np. dla usługi SMB lub FTP)
nmap -p 445 --script smb-vuln-* 10.10.10.10
nmap --script vuln 10.10.10.10

# Zapisanie wyników do wszystkich formatów (XML, normalny, Greppable) na później
nmap -p- -sV -oA wyniki_skanu 10.10.10.10
```

### 5. SQLMap (Automatyczny snajper baz danych)

**Do czego służy?** SQLMap to narzędzie, które wyciąga bazy danych jak magik królika z kapelusza. O ile w naszym labie na `index.php` uczyłeś się SQL Injection z palca, o tyle w prawdziwym życiu, gdy masz np. opóźnienia czasowe (Time-Based Blind SQLi), robienie tego ręcznie zajęłoby Ci tygodnie. Wskazujesz SQLMapowi dziurawy parametr w URL lub pliku żądania, a on sam ustala typ bazy danych (MySQL, PostgreSQL, Oracle), rodzaj podatności i pozwala zrzucić (wydumpować) całe tabele, hasła, a nawet – jeśli uprawnienia na to pozwalają – uzyskać dostęp do powłoki serwera (OS shell).

**Poziomy ryzyka i agresji:**

- `--level` (1-5): Im wyżej, tym więcej miejsc sprawdza (np. na poziomie 3 sprawdza też nagłówek User-Agent, a na 5 wszystko łącznie z ciasteczkami).
    
- `--risk` (1-3): Ryzyko 1 jest bezpieczne. Ryzyko 3 może dodać złośliwe wpisy do bazy lub zepsuć aplikację docelową (odpalaj z głową na produkcji).
    

**Najważniejsze parametry i przykłady użycia:**



```Bash
# Prosty test podatności parametru GET na stronie
sqlmap -u "http://localhost/index.php?id=1"

# Gdy podatność zostanie potwierdzona, zrzucamy listę wszystkich baz danych
sqlmap -u "http://localhost/index.php?id=1" --dbs

# Wybór konkretnej bazy i zrzut listy jej tabel (-D = baza)
sqlmap -u "http://localhost/index.php?id=1" -D lab_db --tables

# Wydobycie (dump) wszystkich danych z tabeli 'users' z bazy 'lab_db'
sqlmap -u "http://localhost/index.php?id=1" -D lab_db -T users --dump

# Atak przez formularz POST (np. logowanie)
sqlmap -u "http://example.com/login.php" --data="username=admin&password=test"

# Automatyzacja: Zgadzaj się na wszystko (--batch) i używaj losowych nagłówków (--random-agent)
sqlmap -u "http://example.com/page?id=1" --batch --random-agent --dbs

# Tunelowanie ruchu przez Tora (żeby nie zjarać swojego IP przy ciężkim ataku)
sqlmap -u "http://example.com/page?id=1" --tor --tor-type=SOCKS5
```

### 6. XSStrike (Chirurg do spraw XSS)

**Do czego służy?** Większość tanich skanerów wysyła na ślepo tysiące popularnych payloadów typu `<script>alert(1)</script>` i modli się, żeby któryś zadziałał. XSStrike robi to inaczej. To inteligentny system, który analizuje dokument (DOM), sprawdza, w jakim kontekście znalazł się Twój input (np. czy wewnątrz atrybutu HTML, czy wewnątrz skryptu JS), i generuje **jeden, idealnie skrojony payload**, który ominie filtry (WAF). Posiada też świetnego fuzzera i potrafi skanować całą stronę w poszukiwaniu ukrytych parametrów, które są podatne na Cross-Site Scripting.

**Najważniejsze parametry i przykłady użycia:**


```Bash
# Podstawowy skan konkretnego parametru w URL
python3 xsstrike.py -u "http://localhost/index2.php?search=test"

# Omijanie WAF (Web Application Firewall) z ustawionym opóźnieniem
python3 xsstrike.py -u "http://example.com/search?q=test" --timeout 5 --delay 2

# Fuzzowanie ukrytych parametrów na stronie (jeśli np. nie wiesz, jak nazywa się pole ukryte)
python3 xsstrike.py -u "http://example.com/page" --fuzzer

# Crawlowanie (przeszukiwanie) całej witryny pod kątem podatności XSS (głębokość = 3)
python3 xsstrike.py -u "http://example.com" --crawl -l 3

# Atakowanie metodą POST (zamiast GET) w formularzu
python3 xsstrike.py -u "http://localhost/index.php" --data "comment=test"
```

### 7. FFUF (Fuzz Faster U Fool)

**Do czego służy?** FFUF to napisany w języku Go, absurdalnie szybki fuzzer aplikacji webowych. Zastąpił starsze narzędzia typu DirBuster czy Dirb. Do czego służy Fuzzing? Bierzesz potężny plik tekstowy (tzw. wordlistę, np. SecLists), w którym są setki tysięcy nazw katalogów, plików, słów kluczowych czy nazw parametrów. FFUF wstawia te słowa po kolei w miejsce słowa klucza `FUZZ` i patrzy, co odpowie serwer. Dzięki niemu znajdziesz ukryte panele logowania (np. `/admin_panel_123`), zapomniane pliki z backupami (np. `/database.sql.bak`) albo niezabezpieczone API.

**Najważniejsze parametry i przykłady użycia:**

```Bash
# Szukanie ukrytych katalogów (np. podmienia słowo FUZZ na 'admin', 'backup', 'login')
# Zwróci wszystko, co ma kod HTTP 200 (OK)
ffuf -u "http://example.com/FUZZ" -w /usr/share/wordlists/dirb/common.txt

# Szukanie konkretnych rozszerzeń plików (np. .php, .txt, .zip)
ffuf -u "http://example.com/FUZZ" -w wordlista.txt -e .php,.txt,.zip

# Fuzzowanie parametrów GET (szukamy ukrytych zmiennych, np. ?debug=1)
ffuf -u "http://example.com/index.php?FUZZ=1" -w parametry.txt

# Ukrywanie śmieci: Wykluczanie wyników na podstawie ilości słów, linii lub rozmiaru. 
# (-fs 42) -> Ignoruj wyniki, w których waga strony to dokładnie 42 bajty (częsty "false positive")
ffuf -u "http://example.com/FUZZ" -w wordlista.txt -fs 42

# Szukanie ukrytych wirtualnych hostów (VHosts/Subdomen) po nagłówkach
ffuf -u "http://example.com" -H "Host: FUZZ.example.com" -w subdomeny.txt -mc 200

# Zwiększanie prędkości (-t to wątki) i zapis do formatu JSON
ffuf -u "http://example.com/FUZZ" -w wordlista.txt -t 100 -o wyniki_ffuf.json
```

### Podsumowanie Rozdziału: Fundamenty, OPSEC i Arsenał Gotowy do Akcji B-)

Uff, dobra panie monter, zdejmujemy na chwilę kominiarki. To był potężny zrzut wiedzy, ale jeśli przebrnąłeś przez to wszystko i postawiłeś system według tych wytycznych, to gratulacje – właśnie przeskoczyłeś 90% "hakerów" z TikToka, którzy odpalają Kali Linuxa na domowym WiFi i dziwią się, że banują im IP.

Zróbmy szybki rachunek sumienia, z czym kończymy ten rozdział:

- **Opancerzony System:** Masz uszczelnionego Debiana (UFW, Fail2ban, Auditd) i zabezpieczony DNS. Nic nie wycieka na zewnątrz bez Twojej wiedzy.
    
- **Prywatność i Trasowanie Ruchu:** Skonfigurowaliśmy Firefoxa tak, by nie sypał logami, ucięliśmy łeb WebRTC i Canvasom. Co więcej, wiesz już czym różni się tryb **Tor-over-VPN** (standardowa tarcza) od **VPN-over-Tor** (omijanie Cloudflare i banów Exit Nodes z obsługą UDP).
    
- **Własny Poligon Doświadczalny:** Postawiliśmy lokalnego laba (LAMP) z aplikacjami `index.php` i `index2.php`, które aż proszą się o wstrzyknięcie złośliwego kodu. Wszystko w pełni bezpiecznie, bez wychodzenia do zewnętrznej sieci.
    
- **Święta Siódemka Audytora:** Masz zainstalowany, zaktualizowany i rozpisany na komendy potężny arsenał:
    
    1. **WhatWeb** – do cichego rozpoznania terenu.
        
    2. **Nikto** – do szukania starych, zapomnianych brudów na serwerze.
        
    3. **Nuclei** – do bezlitosnego egzekwowania najnowszych podatności i CVE.
        
    4. **Nmap** – do mapowania portów i warstwy sieciowej.
        
    5. **SQLMap** – do automatycznego wysysania dziurawych baz danych.
        
    6. **XSStrike** – do precyzyjnego omijania WAF-ów i cięcia XSS-ami jak skalpelem.
        
    7. **FFUF** – do fuzzowania z prędkością światła i odkrywania ukrytych katalogów.
        

**Co dalej wariacie?** Koniec z suchą teorią i konfiguracją. Masz narzędzia, masz wiedzę o ich flagach i masz cel (swój lokalny poligon). W następnym rozdziale odpinamy wrotki, odpalamy terminal i lecimy z praktycznym audytem. Będziemy na żywo fuzować ścieżki, rzucać SQLMapem w parametry GET i parsować wyniki skanów, żeby finalnie zautomatyzować to wszystko przy pomocy AI (Gemini).

Odpalajcie serwery lokalne, róbcie notatki z komend i widzimy się w kolejnej części. Trzymajcie się, cześć! 🥷

_(A jeśli wiedza siadła, przypominam o tabelce z krypto wyżej! Monero zawsze w cenie!)_

wygeneruj mi kod html na podstawie tego tekstu chce go uzyc potem jako tekst do tutoriala na podstronie wlasciwie stworz sam podstrone tutoriala nazwe i inne dobierz na podstawie tekstu 