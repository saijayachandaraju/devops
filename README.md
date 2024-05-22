---
- name: Download deployer from blob storage
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Ensure directory exists
      file:
        path: "/path/to/download"
        state: directory

    - name: Download deployer
      azure_rm_storageblob:
        storage_account: "<storage_account_name>"
        container: "<container_name>"
        blob: "<blob_name>"
        dest: "/path/to/download/deployer"
        mode: "get"
      register: download_status

    - name: Check download status
      debug:
        msg: "Download successful"
      when: download_status.changed

    - name: Handle download failure
      fail:
        msg: "Failed to download deployer from blob storage"
      when: download_status.failed
---
- name: Download and copy deployer to TMS App Server
  hosts: tms_app_server
  tasks:
    - name: Ensure directory exists on TMS App Server
      file:
        path: "<TMDrive>/Programs"
        state: directory
      become: yes

    - name: Download deployer from blob storage
      azure_rm_storageblob:
        storage_account: "<storage_account_name>"
        container: "<container_name>"
        blob: "<blob_name>"
        dest: "/tmp/deployer"
        mode: "get"

    - name: Copy deployer to TMS App Server
      copy:
        src: "/tmp/deployer"
        dest: "<TMDrive>/Programs/deployer"
      become: yes
---
- name: Download, extract, and copy deployer to TMS App Server
  hosts: tms_app_server
  tasks:
    - name: Ensure directory exists on TMS App Server
      file:
        path: "<TMDrive>/Programs"
        state: directory
      become: yes

    - name: Download deployer zip from blob storage
      azure_rm_storageblob:
        storage_account: "<storage_account_name>"
        container: "<container_name>"
        blob: "<blob_name>"
        dest: "/tmp/deployer.zip"
        mode: "get"

    - name: Extract deployer zip file
      unarchive:
        src: "/tmp/deployer.zip"
        dest: "<TMDrive>/Programs"
        remote_src: yes
        extra_opts: [--strip-components=1]
      become: yes

    - name: Remove deployer zip file
      file:
        path: "/tmp/deployer.zip"
        state: absent
      become: yes
---
- name: Update app.config file
  hosts: tms_app_server
  tasks:
    - name: Update JavaHome in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^JavaHome\s*='
        line: 'JavaHome = Java Home Path'
      become: yes

    - name: Update RootDir in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^RootDir\s*='
        line: 'RootDir = Working directory of the program'
      become: yes

    - name: Update Environment in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^Environment\s*='
        line: 'Environment = Environment name (Dev/Test/Prod)'
      become: yes

    - name: Update UserName in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^UserName\s*='
        line: 'UserName = TM User name to use and invoke API call'
      become: yes

    - name: Update Password in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^Password\s*='
        line: 'Password = Password of the TM User mentioned above'
      become: yes

    - name: Update smtp.Host in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^smtp.Host\s*='
        line: 'smtp.Host = SMTP Host server'
      become: yes

    - name: Update smtp.From in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^smtp.From\s*='
        line: 'smtp.From = From address of alert notification email'
      become: yes

    - name: Update smtp.To in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^smtp.To\s*='
        line: 'smtp.To = Recepient list of alert notification email'
      become: yes

    - name: Update smtp.Cc in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^smtp.Cc\s*='
        line: 'smtp.Cc = Recepient list of alert notification email'
      become: yes

    - name: Update smtp.EnableEmail in app.config
      lineinfile:
        path: "<TMDrive>/Programs/CISAdapterMonitor/app.config"
        regexp: '^smtp.EnableEmail\s*='
        line: 'smtp.EnableEmail = True for receiving email alerts/False for no email alerts'
      become: yes
---
- name: Update ServerAdapterList.csv file
  hosts: tms_app_servers
  vars:
    server_instance_map:
      - { Server: "tsbf030202001", InstanceID: "RR", PortType: "ShipmentConsolidation", CustomerName: "CHEP" }
      - { Server: "tsbf030202001", InstanceID: "TMAPI", PortType: "TransportationManager", CustomerName: "CHEP" }
      - { Server: "tsbf030202001", InstanceID: "TMAPI2", PortType: "TransportationManager2", CustomerName: "CHEP" }
      - { Server: "tsbf030201001", InstanceID: "RR", PortType: "ShipmentConsolidation", CustomerName: "CHEP" }
      - { Server: "tsbf030201001", InstanceID: "TMAPI", PortType: "TransportationManager", CustomerName: "CHEP" }
  tasks:
    - name: Generate ServerAdapterList.csv file
      template:
        src: server_adapter_list.csv.j2
        dest: "<TMDrive>/Programs/ServerAdapterList.csv"
