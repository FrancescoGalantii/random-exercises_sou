# **`spiegazione progetto`**
l'obiettivo di questo progetto era prender confidenza con i template Jinja e vedere come essi possano aiutare nella creazione di file configurabili dinamicamente.

## spiegazione playbook

**`primo step`**; aggiungere aggiungere in append sul file `/etc/security/limits.conf` alcuni settings per un’utente. 

• In ambiente di produzione dobbiamo imporre un numero massimo di file aperti pari a 10000, mentre in ambiente di collaudo e sviluppo 1000
environment: "prod" 
    
    user_limits:
      prod: 10000
      dev: 1000
      test: 1000

• Supponiamo che in `/etc/security/access.conf` ci sia un’ultima riga che impedisce l’accesso agli utenti non esplicitamente autorizzati (“- : ALL : ALL”). 

• Creare un playbook Ansible che aggiunga una lista di utenti in whitelist anteponendosi a tale riga (hint: utilizzare l’opzione insertbefore del modulo blockinfile).
sono andato perciò prima a specificare gli utenti:

    user_name: utenti 
    whitelist_users:
      - francesco
      - lorenzo
      - giosue
per poi configurare la `whitelist`:
      
    tasks:
      - name: Configura whitelist utenti in access.conf
        blockinfile:
          path: /etc/security/access.conf
          block: |
            {% for user in whitelist_users %}
            + : {{ user }} : ALL
            {% endfor %}
          insertbefore: '^-\s*:\s*ALL\s*:\s*ALL'
          state: present
        tags: whitelist
• [OPZIONALE]; Trovare un possibile utilizzo dei Jinja templates per la parametrizzazione di almeno uno dei seguenti oggetti:

- **`Dockerfile`**
- **`File yaml rappresentante un pod o un deployment Kubernetes`**
- **`Virtual host di apache`**

Nel mio caso ho deciso di utilizzare i templates Jinja per la parametrizzazione di un
• **`pod.yaml.j2`**

    apiVersion: v1
    kind: Pod
    metadata:
      name: {{ pod_spec.pod_name }}
    spec:
      containers:
      - name: {{ pod_spec.container_name }}
        image: {{ pod_spec.container_image }}
        ports:
        - containerPort: {{ pod_spec.container_port }}
richiamando il pod_spec definito all'interno del playbook

    pod_spec:
      pod_name: "my-pod"
      container_name: "my-container"
      container_image: "nginx:latest"
      container_port: 80
• e per un host virtuale apache --> **`virtualhost.conf.j2`**

    <VirtualHost *:80>
        ServerName {{ apache_config.server_name }}
        DocumentRoot {{ apache_config.document_root }}

        <Directory "{{ apache_config.document_root }}">
        AllowOverride All
        Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
per far si che tutto funzioni ho scaricato apache sulla macchina virtuale, creata tramite `Vagrantfile`, attraverso i seguenti comandi:

    sudo dnf update -y
    sudo dnf install httpd -y
    sudo systemctl start httpd
    sudo systemctl enable httpd
se non si ha apache scaricato all'esecuzione del playbook si avra il seguente errore:

fatal: [default]: FAILED! => {"changed": false, "checksum": "d2e92065bf18e319e7e089dcd5d41c5cad9efcd2", "msg": "Destination directory /etc/httpd/conf.d does not exist"

Arrivati a questo punto per verificare che tutto sia andato a buon fine lanciare i seguenti comandi:

• `vagrant`; ssh per entrare all'interno della vm

• `cd /etc/security`; per accedere alla directory contenente il file access.conf

• `cat access.conf`; per controllare se sono stati aggiunti in append gli utenti della whitelist

• `cat pod.yaml`; per vedere se è stato creato il pod 

• `cd /etc/httpd/conf.d/`; per entrare nella cartella di destinazione del host apache

• `cat file_host.com.conf`; per vedere se è stato creato l'host apache.



