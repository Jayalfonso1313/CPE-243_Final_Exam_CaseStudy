# CPE-243_Final_Exam_CaseStudy


---
- hosts: all
  become: true
  vars:
    openssl_csr:
      path: /etc/ssl/certs/ca.csr
      privatekey_path: /etc/ssl/private/ca.key
      common_name: JAY
      country_name: PH
      state_or_province_name: Metro Manila
      locality_name: Quezon City
      organization_name: TIPQC
      email_address: qjjalfonso@tip.edu.ph
  
  tasks:
    - name: update (Ubuntu)
      tags: always
      apt:
        update_cache: yes
      when: ansible_distribution == 'Ubuntu'

    - name: update (CentOS)
      tags: always
      yum:
        name: "*"
        state: latest
      when: ansible_distribution == 'CentOS'

    - name: Install OpenSSL
      apt:
        name: openssl
        state: present
      when: ansible_distribution == 'Ubuntu'

    - name: Install OpenSSL
      yum:
        name: openssl
        state: present
      when: ansible_distribution == 'CentOS'

    - name: Generating CA private key
      openssl_privatekey:
        path: "{{ openssl_csr.privatekey_path }}"
        size: 4096
      register: ca_key
                
    - name: Generating CSR
      openssl_csr:
        path: "{{ openssl_csr.path }}"
        privatekey_path: "{{ openssl_csr.privatekey_path }}"
        common_name: "{{ openssl_csr.common_name }}"
        country_name: "{{ openssl_csr.country_name }}"
        state_or_province_name: "{{ openssl_csr.state_or_province_name }}"
        locality_name: "{{ openssl_csr.locality_name }}"
        organization_name: "{{ openssl_csr.organization_name }}"
        email_address: "{{ openssl_csr.email_address }}"
      register: ca_csr

    - name: Generating CA certificate
      openssl_certificate:
        path: /etc/ssl/certs/ca.crt
        privatekey_path: "{{ openssl_csr.privatekey_path }}"
        csr_path: "{{ openssl_csr.path }}"
        provider: selfsigned
        force: true

    - name: Copying the CA certificates (Ubuntu)
      fetch:
        src: /etc/ssl/certs/ca-certificates.crt
        dest: /tmp/ca_certificate/
        flat: yes
      delegate_to: localhost
      when: ansible_distribution == 'Ubuntu'

    - name: Copying the CA certificates (CentOS)
      fetch:
        src: /etc/pki/tls/certs/ca-bundle.crt
        dest: /tmp/ca_certificate/
        flat: yes
      delegate_to: localhost
      when: ansible_distribution == 'CentOS'
