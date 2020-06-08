# AWS Linux

## Instalamos Servidor en AWS
* Region: US East (N. Virginia) -> us-east-1
* VPC: formacion - CIDR Block: 10.10.0.0/16
* Subnet: formacion - CIDR Block: 10.10.0.0/16
* Internet gateway: formacion -> atachada a la VPC
* Public subnet route table: formacion
<pre>
0.0.0.0/0 igw
</pre>
* Security Group: formacion -> atachada a la VPC
<pre>
Inboud rules:
Type: SSH
Procotol: TCP
Port Range: 22
Source: 0.0.0.0/0
</pre>
* Instancias EC2:    
  * formacion_rhel8 -> RHEL-8.2.0_HVM-20200423-x86_64-0-Hourly2-GP2 (ami-098f16afa9edf40be)
    * t2.micro
 
  * formacion_sles15 -> suse-sles-15-sp1-v20200226-hvm-ssd-x86_64 (ami-0068cd63259e9f24c)
    * t2.micro
* Conexión:
<pre>
+formacion_rhel8
ssh -i /home/renzo/aws_linux/aws_keys/formacion.pem ec2-user@ec2_rhel8

+formacion_sles15
ssh -i /home/renzo/aws_linux/aws_keys/formacion.pem ec2-user@ec2_sles15
</pre>

## Configurar Hostname
* Hostname:
<pre>
hostnamectl

hostnamectl set-hostname ec2_rhel8
ls -l /etc/hostname

hostnamectl set-hostname ec2_sles15
ls -l /etc/HOSTNAME
</pre>

## Configurar fichero /etc/hosts
</pre>
ip a s

cp -p /etc/hosts /etc/hosts.`date +%Y%m%d`
diff /etc/hosts /etc/hosts.`date +%Y%m%d`
4d3
< 10.10.130.90 ec2_rhel8.pepe.com ec2_rhel8

cp -p /etc/hosts /etc/hosts.`date +%Y%m%d`
diff /etc/hosts /etc/hosts.`date +%Y%m%d`
27,28d26
< 
< 10.10.14.176	ec2_sles15.pepe.com ec2_sles15

hostname
hostname -f
</pre>

## SELINUX
* Saber estado:
<pre>
sestatus

+En el ami de SLES15 por defecto el selinux esta deshabilitado.
</pre>

* Deshabilitar SELINUX
<pre>
+En caliente: setenforce 0
++permissive - SELinux prints warnings instead of enforcing.

+Persistente: /etc/selinux/config
cp -p /etc/selinux/config /etc/selinux/config`date +%Y%m%d`
sed 's/^SELINUX=.*$/SELINUX=disabled/g' /etc/selinux/config
sed -i 's/^SELINUX=.*$/SELINUX=disabled/g' /etc/selinux/config
diff /etc/selinux/config /etc/selinux/config`date +%Y%m%d`
++disabled - No SELinux policy is loaded.
</pre>

## Configurar Timezone & NTP
* Info:
 * https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/set-time.html

* Timezone:
<pre>
ls -lrt /usr/share/zoneinfo/Europe/Madrid
ls -l /etc/localtime
date

ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
date

+++ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
</pre>

* NTP:
<pre>
http://support.ntp.org/bin/view/Servers/WebHome
http://support.ntp.org/bin/view/Servers/NTPPoolServers
rpm -qa | grep chrony

+NTP status:
chronyc sources -v
cat /etc/chrony.conf  | egrep -v "^#|^$"
cp -p /etc/chrony.conf /etc/chrony.conf.`date +%Y%m%d`

systemctl restart chronyd
systemctl is-enabled chronyd
</pre>

## Servicios
* Listar todos los servicios:
<pre>
systemctl list-unit-files

systemctl -la
</pre>

* Estado de servicios:
<pre>
systemctl status chronyd
</pre>

* Parar/Reiniciar/Iniciar:
<pre>
systemctl stop chronyd
systemctl start chronyd
systemctl restart chronyd
</pre>

* Validar si está configurado en el arranque del sistema, habilitar/deshabilitar:
<pre>
systemctl is-enabled chronyd
systemctl disable chronyd
systemctl enable chronyd
</pre>
## Parámetros de kernel
* Configurar parámetros de kernel
<pre>
/etc/sysctl.conf

sysctl -p
sysctl -a
</pre>

## Límites
<pre>
/etc/security/limits.conf

/etc/security/limits.d/
</pre>

## Gestión de Paquetes
* Paquetes básicos o recomendados a instalar en una instalación de Linux (telnet, unzip…)
<pre>
-------------------------------------------------------------------------------------------------------------
+RHEL:
yum clean all                            dnf clean all
yum repolist                             dnf repolist
cd /etc/yum.reposd                       # <--- ubicacion de repositorios configurados en el servidor
yum list                                 dnf list
yum search                               dnf search 
yum provides '*/ls'                      dnf provides */ls
yum info                                 dnf info vim

yum install telnet unzip zip wget        dnf install telnet unzip zip wget 
yum install xorg-x11-xauth               dnf install xorg-x11-xauth 

yum remove XXXX                          dnf remove XXXX

-------------------------------------------------------------------------------------------------------------
+SLES:
zypper install telnet unzip zip wget
zypper install xorg-x11-xauth

zypper ls
zypper se
zypper search --provides '*/ls'

-------------------------------------------------------------------------------------------------------------
+RPM:
rpm -qa
rpm -qi
rpm -ql bash-4.4.19-10.el8.x86_64
rpm -qf /usr/share/man/man1/times.1.gz

-------------------------------------------------------------------------------------------------------------
</pre>

## Filesystems
* Configurar filesystem
  * Los Instance Store Volumes no pueden ser detenidos, si el sistema falla los datos se pierden.

* Elastic Block Store
  * volumes
  * /dev/sdb
  * fdisk -l | grep ^Disk
  * lsblk -f

* yum install lvm2 (+En el AMI de sles15 ya viene por defecto instalado)

## Swap
* Tamaño:
<pre>
SWAP
Configurar swap según recomendaciones de SO:
                - RHEL: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-swapspace
                - Ubuntu: https://help.ubuntu.com/community/SwapFaq?_ga=2.73427474.1003327733.1559477591-305631208.1559477591#How_much_swap_do_I_need.3F

AWS:
    - https://aws.amazon.com/es/premiumsupport/knowledge-center/ec2-memory-partition-hard-drive/

#####SWAP
#2GB or less  Twice the installed RAM
#> 2GB - 8GB  The same amount of RAM
#> 8GB - 64GB At least 4GB
#> 64GB or more At least 4GB
</pre>

* Configurar swap
<pre>
mkswap /dev/vg_data/lv_swap

cp -p /etc/fstab /etc/fstab.`date +%Y%m%d`
/dev/vg_data/lv_swap                    swap                    swap      defaults        0 0

swapon -a
</pre>

## Configurar las Xs
* Paquete a isntalar:
<pre>
+RHEL:
yum install xorg-x11-xauth
wget https://rpmfind.net/linux/centos/8.1.1911/PowerTools/x86_64/os/Packages/xorg-x11-apps-7.7-21.el8.x86_64.rpm
+SLES:
zypper install xorg-x11-xauth
wget https://download.opensuse.org/repositories/openSUSE:/Leap:/15.1/standard/x86_64/xclock-1.0.7-lp151.2.3.x86_64.rpm
</pre>

* Tener el x11 forwarding habiltado:
<pre>
cat /etc/ssh/sshd_config| grep -i ^X11Forwarding
X11Forwarding yes

sudo su -
touch .Xauthority
xauth add $(xauth -f /home/ec2-user/.Xauthority list|tail -1)
</pre>
