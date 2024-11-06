Als erstes einen Server bereitstellen, wo via SSH mit dem Ansible verbunden ist. Dort kommt dann MariaDB drauf.

Auf der Ansible VM SSH key erstelle:
- Im ordner .ssh `ssh-keygen`
- Key kopieren und auf dem anderen Server bereittstellen. Im authorized_keys hinterlegen (unter den anderen reinkopieren)
- Hostkey Check disablen auf dem Ansible Server -> export ANSIBLE_HOST_KEY_CHECKING=false

#### Schritt 1: Erstelle das Projektverzeichnis und die Rollenstruktur

1. Erstelle ein Verzeichnis für dein Ansible-Projekt:
    ```bash
    mkdir mysql_ansible_project
    cd mysql_ansible_project
    ```

2. Erstelle eine Verzeichnisstruktur für die `mysql`-Rolle:
    ```bash
    mkdir -p roles/mysql/{tasks,handlers,templates,defaults}
    ```

Diese Struktur ist nach Ansible-Standards für Rollen aufgebaut, sodass jede Rolle ihre Aufgaben (`tasks`), Handler (`handlers`), Vorlagen (`templates`) und Standardvariablen (`defaults`) klar organisiert hat.

#### Schritt 2: Erstelle das Haupt-Playbook (`mysql_playbook.yml`)

Dieses Playbook definiert, dass die `mysql`-Rolle auf den definierten Zielservern ausgeführt wird:

1. Erstelle die Datei `mysql_playbook.yml`:
    ```bash
    touch mysql_playbook.yml
    ```

2. Öffne die Datei `mysql_playbook.yml` in einem Texteditor (z.B. `nano`, `vim` oder VS Code) und füge Folgendes ein:

    ```yaml
    ---
    - hosts: all
      become: true
      roles:
        - mysql
    ```

Hier definieren wir:
- `hosts: all` – das Playbook wird auf allen Ziel-Hosts ausgeführt, die du in deiner Ansible-Inventar-Datei angibst.
- `become: true` – das Playbook wird mit erhöhten Berechtigungen (Root) ausgeführt.
- `roles: mysql` – führt die `mysql`-Rolle aus, die wir gleich erstellen.

#### Schritt 3: Erstelle die Aufgaben in der Rolle (`roles/mysql/tasks/main.yml`)

In der Datei `tasks/main.yml` definieren wir, wie MariaDB installiert und konfiguriert wird.

1. Erstelle die Datei `roles/mysql/tasks/main.yml`:
    ```bash
    touch roles/mysql/tasks/main.yml
    ```

2. Füge folgenden Inhalt ein:

    ```yaml
    ---
    # Installiere die MariaDB-Pakete
    - name: Installiere MariaDB-Pakete
      ansible.builtin.package:
        name: "{{ mariadb_packages }}"
        state: present

    # Konfiguriere MariaDB mit einem Jinja-Template
    - name: Konfiguriere MariaDB
      ansible.builtin.template:
        src: 50-server.cnf.j2
        dest: /etc/mysql/mariadb.conf.d/50-server.cnf
        owner: root
        group: root
        mode: '0644'
      notify: Restart MariaDB

    # Stelle sicher, dass der MariaDB-Dienst läuft und beim Booten gestartet wird
    - name: Stelle sicher, dass MariaDB läuft und aktiviert ist
      ansible.builtin.service:
        name: mariadb
        state: started
        enabled: true
    ```

Hier passiert Folgendes:
- Wir installieren MariaDB-Pakete.
- Wir kopieren ein Konfigurations-Template (`50-server.cnf.j2`) auf den Server.
- Wir stellen sicher, dass MariaDB gestartet ist und beim Booten automatisch startet.
- Der `notify`-Abschnitt löst den `Restart MariaDB`-Handler aus, wenn das Template eine Änderung am Server bewirkt.

#### Schritt 4: Definiere den Handler (`roles/mysql/handlers/main.yml`)

Der Handler wird ausgelöst, wenn sich die Konfiguration ändert und der Dienst neu gestartet werden muss.

1. Erstelle die Datei `roles/mysql/handlers/main.yml`:
    ```bash
    touch roles/mysql/handlers/main.yml
    ```

2. Füge folgenden Inhalt ein:

    ```yaml
    ---
    - name: Restart MariaDB
      ansible.builtin.service:
        name: mariadb
        state: restarted
    ```

Der Handler wird nur ausgeführt, wenn die Konfigurationsdatei verändert wurde und ein Neustart des Dienstes erforderlich ist.

#### Schritt 5: Erstelle das Jinja-Template für die Konfiguration (`roles/mysql/templates/50-server.cnf.j2`)

Das Template definiert die Konfigurationsparameter für MariaDB.

1. Erstelle die Datei `roles/mysql/templates/50-server.cnf.j2`:
    ```bash
    touch roles/mysql/templates/50-server.cnf.j2
    ```

2. Füge folgende Konfiguration ein:

    ```ini
    # MariaDB Konfiguration
    [mysqld]
    port = {{ mysql_port }}
    bind-address = {{ mysql_bind_address }}
    ```

Hier verwenden wir `{{ mysql_port }}` und `{{ mysql_bind_address }}` als Platzhalter. Diese werden später durch Werte ersetzt, die wir in `defaults/main.yml` definieren.

#### Schritt 6: Definiere Standardwerte für Variablen (`roles/mysql/defaults/main.yml`)

In `defaults/main.yml` legen wir die Standardwerte für die Variablen fest, die im Template verwendet werden.

1. Erstelle die Datei `roles/mysql/defaults/main.yml`:
    ```bash
    touch roles/mysql/defaults/main.yml
    ```

2. Füge folgende Standardwerte ein:

    ```yaml
    ---
    mariadb_packages:
      - mariadb-server
      - mariadb-client

    mysql_port: 3306
    mysql_bind_address: 0.0.0.0
    ```

Hier geben wir an:
- `mariadb_packages`: Die Pakete, die installiert werden sollen.
- `mysql_port`: Der Port, auf dem MariaDB lauscht (Standard 3306).
- `mysql_bind_address`: Die IP-Adresse, an die MariaDB gebunden wird (Standard `0.0.0.0` für alle Netzwerkschnittstellen).

#### Schritt 7: Hosts File erstellen

Hosts datei erstellen und folgendes hinterlegen:
```bash
[database_server]
webserver ansible_host=172.22.141.206 ansible_user=ubuntu
```
Name des servers und IP hinterlegen, plus root user.

#### Schritt 8:Führe das Playbook aus

Jetzt, da alles eingerichtet ist, kannst du das Playbook ausführen:

1. Stelle sicher, dass deine `hosts`-Datei den Zielserver definiert.
2. Führe das Playbook aus:

    ```bash
    ansible-playbook -i hosts mysql_playbook.yml
    ```

### Zusammenfassung

Das Playbook:
1. Installiert MariaDB.
2. Konfiguriert MariaDB mit den Werten aus der `defaults/main.yml`.
3. Startet MariaDB und stellt sicher, dass es beim Systemstart geladen wird.
4. Führt den Handler `Restart MariaDB` aus, wenn Änderungen an der Konfiguration vorgenommen werden.

Das Ergebnis ist eine wiederverwendbare Ansible-Rolle, die MariaDB installiert und nach deinen Vorgaben konfiguriert.