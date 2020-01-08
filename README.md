# ansiblefolder
---
# wordpress

- name: Baixa o WordPress
  get_url:
    url=https://wordpress.org/latest.tar.gz
    dest=/tmp/wordpress.tar.gz
    validate_certs=no

- name: Descompacta o WordPress
  unarchive: src=/tmp/wordpress.tar.gz dest=/var/www/ copy=no
  become: yes

- name: Atualiza o site Apache padrao
  lineinfile:
    dest=/etc/apache2/sites-enabled/000-default.conf
    regexp="(.)+DocumentRoot /var/www/html"
    line="DocumentRoot /var/www/wordpress"
  notify:
    - reinicia apache

- name: Renomeia o arquivo de configuracao de exemplo
  command: mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php creates=/var/www/wordpress/wp-config.php
  become: yes

- name: Atualiza a configuracao WordPress
  lineinfile:
    dest=/var/www/wordpress/wp-config.php
    regexp="{{ item.regexp }}"
    line="{{ item.line }}"
  with_items:
    - {'regexp': "define\\('DB_NAME', '(.)+'\\);", 'line': "define('DB_NAME', '{{wp_mysql_db}}');"}
    - {'regexp': "define\\('DB_USER', '(.)+'\\);", 'line': "define('DB_USER', '{{wp_mysql_user}}');"}
    - {'regexp': "define\\('DB_PASSWORD', '(.)+'\\);", 'line': "define('DB_PASSWORD', '{{wp_mysql_password}}');"}
  become: yes
