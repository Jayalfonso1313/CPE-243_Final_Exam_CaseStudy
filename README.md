# CPE-243_Final_Exam_CaseStudy

---
- name: Patch Management
  hosts: all
  become: yes

  tasks:
    - name: Update package cache (Ubuntu)
      apt:
        update_cache: yes
      when: ansible_distribution == 'Ubuntu'

    - name: Install security updates (Ubuntu)
      apt:
        name: '*'
        state: latest
        upgrade: yes
        autoremove: yes
        autoclean: yes
      when: ansible_distribution == 'Ubuntu'

    - name: Update package cache (CentOS)
      yum:
        name: '*'
        state: latest
      when: ansible_distribution == 'CentOS'

    - name: Install security updates (CentOS)
      yum:
        name: '*'
        state: latest
        security: yes
      when: ansible_distribution == 'CentOS'
