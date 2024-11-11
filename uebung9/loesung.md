- name: Testplaybook
  hosts: localhost
  vars:
    configfile: myapp/myapp.conf
  connection: local
  tasks:
    - name: Make backup file
      shell: "cp {{ configfile }} {{ configfile }}.{{ ansible_date_time.iso8601_basic_short }}"
      register: backup_creation
      changed_when: true

    # Hier kommt was auch immer getan werden muss mit dem configfile

    # Cleanup Task: Lösche alle Backups außer die letzten 3
    - name: Get the list of backup files ordered by timestamp
      shell: "ls -1t {{ configfile }}.*"
      register: lsoutput
      check_mode: no
      ignore_errors: true

    # Debug-Task: zeigt alle Felder von lsoutput an
    - debug:
        msg: "{{ lsoutput }}"

    # Lösche alle Dateien außer den letzten 3
    - name: Remove files except the last 3
      file:
        path: "{{ item }}"
        state: absent
      loop: "{{ lsoutput.stdout_lines[3:] }}"
      when: lsoutput.stdout_lines | length > 3

    # Debug-Task zur Anzeige der gelöschten Dateien
    - name: Debug removed files
      debug:
        msg: "Deleted backup file: {{ item }}"
      loop: "{{ lsoutput.stdout_lines[3:] }}"
      when: lsoutput.stdout_lines | length > 3
