# webchat

# Instalacão
<pre>
cat <<EOF > /etc/apt/source.list
deb http://http.us.debian.org/debian/ stretch main
deb-src http://http.us.debian.org/debian/ stretch main
deb http://security.debian.org/debian-security stretch/updates main
deb-src http://security.debian.org/debian-security stretch/updates main
# stretch-updates, previously known as 'volatile'
deb http://http.us.debian.org/debian/ stretch-updates main
deb-src http://http.us.debian.org/debian/ stretch-updates main
EOF

apt-get update 
apt-get install -y locales-all
localectl set-locale LANG=pt_BR.UTF-8 LC_CTYPE=pt_BR.UTF-8
source /etc/default/locale
export LANG=pt_BR.UTF-8 LC_CTYPE=pt_BR.UTF-8 LC_ALL=pt_BR.UTF-8

apt-get install -y sudo net-tools wget ca-certificates git sqlite bash libmariadb-dev libpq-dev build-essential checkinstall gcc g++ python3-setuptools libmagic-dev libev-dev libgcrypt20-dev libxml2-dev libxslt1-dev libffi-dev fontconfig cython mariadb-client postgresql-client libmariadbclient-dev libdbus-glib-1-2 screen locales-all at nginx

apt-get install -y libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev
{
 apt install fonts-freefont-ttf
} ||
{
apt install ttf-freefont 
}

echo "www-data    ALL=NOPASSWD: /usr/local/webchat/bin/sudo.sh" > /etc/sudoers.d/webchat

apt install python3-pip


git clone https://github.com/thiagosm/webchat.git /usr/local/webchat

cd /usr/local/webchat/
pip3 install -r requirements.txt 

mkdir data
mkdir sessions

python3.5 manage.py makemigrations
python3.5 manage.py migrate
python3.5 manage.py loaddata user
python3.5 manage.py loaddata source
python3.5 manage.py loaddata group
python3.5 manage.py loaddata script
python3.5 manage.py loaddata menu
python3.5 manage.py loaddata menuitem
python3.5 manage.py loaddata filter


cat <<EOF > /etc/nginx/sites-enabled/webchat
server {
    listen 7000;
    server_name _;
    root /var/www;
    index index.html index.htm;
    client_max_body_size 100M;
	location /static { alias /usr/local/webchat/static; }
	rewrite ^/$  	/admin/;    
	location / {
	    root /var/www;
	    proxy_pass http://unix:/run/gunicorn/webchat.socket;
	    include proxy_params;
	    access_log /var/log/nginx/webchat_access.log;
	    error_log /var/log/nginx/webchat_error.log;
	}
}
EOF


cp /usr/local/webchat/install/gunicorn_webchat.socket /etc/systemd/system/
cp /usr/local/webchat/install/gunicorn_webchat.service /etc/systemd/system/
cp /usr/local/webchat/install/webchat@.service /etc/systemd/system/
cp /usr/local/webchat/install/gunicorn.conf /etc/tmpfiles.d/

systemctl enable gunicorn_webchat.socket
systemctl enable gunicorn_webchat.service
systemctl enable nginx
systemctl start gunicorn_webchat.socket
systemctl start gunicorn_webchat.service
systemctl stop nginx
systemctl start nginx 

</pre>

# Contato
suenia@sgp.net.br / 55 83 99606-6939 <br/>
suporte@sgp.net.br
