#
#Limpieza:
#  podman image rm lucianoepc/networking-tools:1.0-x64-alp3.20
#  podman image prune 
#
#Ejemplo del comando de creación/compilación de la imagen:
#  cd ~/code/own1/files/containerfiles
#  podman build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/networking-tools:1.0-x64-alp3.20 ./images/10_devtools/01_networking/networking-tools/1.0-x64-alp3.20/
#  podman run -it --rm --name test01 lucianoepc/networking-tools:1.0-x64-alp3.20
#

#-------------------------------------------------------------------------------------
# Layer final
#-------------------------------------------------------------------------------------
FROM alpine:3.19.1

#1. Parametros de entrada
ARG UID
ARG GID

#2. Constantes
ARG USER_NAME=appuser
ARG HOME_PATH=/home/appuser
ARG TLS11_ENABLE=0
ARG TLS11_CONFIG='\n[ default_conf ]\nssl_conf = ssl_sect\n\n[ssl_sect]\nsystem_default = system_default_sect\n\n[system_default_sect]\nMinProtocol = TLSv1.1\nCipherString = DEFAULT@SECLEVEL=0\n'

#3. Crear el usuario/Grupo para desarrollo e instalar git/sudo
#   > Alpine 3.19 usa OpenSSL 3.0.X la cual por defecto no soporta TLS 1.0, TLS 1.1, TLS 1.2
RUN apk --no-cache add bash git sudo curl \
    busybox-extras openssh-client  \
    #bind-tools net-tools netcat-openbsd iputils tcpdump nmap nmap-scripts nmap-ncat \
    && curl -O https://download.microsoft.com/download/b/9/f/b9f3cce4-3925-46d4-9f46-da08869c6486/msodbcsql18_18.0.1.1-1_amd64.apk \
    && curl -O https://download.microsoft.com/download/b/9/f/b9f3cce4-3925-46d4-9f46-da08869c6486/mssql-tools18_18.0.1.1-1_amd64.apk \
    && apk add --allow-untrusted msodbcsql18_18.0.1.1-1_amd64.apk \
	&& apk add --allow-untrusted mssql-tools18_18.0.1.1-1_amd64.apk \
	&& rm -f msodbcsql18_18.0.1.1-1_amd64.apk mssql-tools18_18.0.1.1-1_amd64.apk \
    && addgroup -g $GID ${USER_NAME} \
    && adduser -D -u $UID -G ${USER_NAME} -h ${HOME_PATH} -s /bin/bash ${USER_NAME} \
    && mkdir -p ${HOME_PATH} \
    && chown $UID:$GID ${HOME_PATH} \
    #&& echo "${USER_NAME} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \
    && if [ ${TLS11_ENABLE} -eq 0 ]; then \
        sed -i 's/openssl_conf = openssl_init/openssl_conf = default_conf/g' /etc/ssl/openssl.cnf; \
        printf "${TLS11_CONFIG}" >> /etc/ssl/openssl.cnf; \
    fi


#4. Descargar archivos de usuario
USER ${UID}:${GID}
RUN git clone https://github.com/lestebanpc/dotfiles.git ${HOME_PATH}/.files

#5. Instalar paquetes basicos (openssl, fzf, vim) y comandos basicos
#   > Parametros de '00_setup_summary.bash':
#     01> Tipo de invocación (sin menu): 1/2 (sin usar un menu interactivo/no-interactivo)
#     02> Opciones de menu a ejecutar: entero positivo.
#     03> ID de los repositorios de comandos a configurar, separados por coma. Si no desea configurarse ninguno envie "EMPTY".
#     04> ID de los paquetes del repositorio del SO, separados por coma, a instalar si elige la opcion de menu 1024. Si desea usar el los los paquetes por defecto envie "EMPTY".
#         Los paquetes por defecto que son: Curl, UnZip, OpenSSL y Tmux.
#     05> Ruta base del home del usuario al cual se configurara su profile y donde esta el repositorio git. Este valor se obtendra segun orden prioridad:
#         - El valor especificado como argumento del script de instalación (debe ser diferente de vacio o "EMPTY")
#         - El valor ingresado en el archivo de configuracion ".config.bash" (debe ser diferente de vacio)
#         - Si ninguno de los anteriores se establece, se la ruta sera calculado en base de la ruta del script de instalación y el nombre del repositorio 'g_repo_name'.
#         - Si no se puede cacluar este valor, se detendra el proceso de instalación/actualización
#     06> Nombre del repositorio git o la ruta relativa del repositorio git respecto al home al cual se desea configurar el profile del usuario.
#         Este valor se obtendra segun orden prioridad:
#         - El valor especificado como argumento del script de instalación (debe ser diferente de vacio o "EMPTY")
#         - El valor ingresado en el archivo de configuracion ".config.bash" (debe ser diferente de vacio)
#         - Si ninguno de los anteriores se establece, se usara el valor '.files'.
#     07> Ruta donde se descargaran los programas (de repositorios como github). Si se envia vacio o EMPTY se usara el directorio predeterminado 
#         "/var/opt/tools" o "~/tools".
#     08> Ruta base donde se almacena los comandos ("CMD_PATH_BASE/bin"), archivos man1 ("CMD_PATH_BASE/man/man1") y fonts ("CMD_PATH_BASE/share/fonts").
#         Si se envia vacio o EMPTY se usara el directorio predeterminado.
#            > Comandos      : "/usr/local/bin"      (para todos los usuarios) y "~/.local/bin"         (solo para el usuario actual)
#            > Archivos man1 : "/usr/local/man/man1" (para todos los usuarios) y "~/.local/man/man1"    (solo para el usuario actual)
#            > Archivo fuente: "/usr/share/fonts"    (para todos los usuarios) y "~/.local/share/fonts" (solo para el usuario actual)
#     09> Ruta de archivos temporales. Si se envia vacio o EMPTY se usara el directorio predeterminado.
#     10> El estado de la credencial almacenada para el sudo.
#     11> Flag '0' para limpiar el cache de paquetes del sistema operativo. Caso contrario, use 1.
#     12> Actualizar los paquetes del SO. Por defecto es 1 (false), si desea actualizar use 0.
#  > Opción de menu a usar: Paquetes basicos 'openssl,fzf' (1024) + VIM (32768) 
USER 0:0
RUN bash ${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash 2 33792 'jq,yq,fd,bat,ripgrep,delta,oh-my-posh,eza,zoxide' 'openssl,fzf' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 0 1

#6. Configurar el profile del usuario como Editor (incluyendo a VIM como Editor)
#   > Parametros de '00_setup_summary.bash':
#     01> Tipo de invocación (sin menu): 1/2 (sin usar un menu interactivo/no-interactivo)
#     02> Opciones de menu a ejecutar: entero positivo.
#     03> ID de los repositorios de comandos a configurar, separados por coma. Si no desea configurarse ninguno envie "EMPTY".
#     04> ID de los paquetes del repositorio del SO, separados por coma, a instalar si elige la opcion de menu 1024. Si desea usar el los los paquetes por defecto envie "EMPTY".
#         Los paquetes por defecto que son: Curl, UnZip, OpenSSL y Tmux.
#     05> Ruta base del home del usuario al cual se configurara su profile y donde esta el repositorio git. Este valor se obtendra segun orden prioridad:
#         - El valor especificado como argumento del script de instalación (debe ser diferente de vacio o "EMPTY")
#         - El valor ingresado en el archivo de configuracion ".config.bash" (debe ser diferente de vacio)
#         - Si ninguno de los anteriores se establece, se la ruta sera calculado en base de la ruta del script de instalación y el nombre del repositorio 'g_repo_name'.
#         - Si no se puede cacluar este valor, se detendra el proceso de instalación/actualización
#     06> Nombre del repositorio git o la ruta relativa del repositorio git respecto al home al cual se desea configurar el profile del usuario.
#         Este valor se obtendra segun orden prioridad:
#         - El valor especificado como argumento del script de instalación (debe ser diferente de vacio o "EMPTY")
#         - El valor ingresado en el archivo de configuracion ".config.bash" (debe ser diferente de vacio)
#         - Si ninguno de los anteriores se establece, se usara el valor '.files'.
#     07> Ruta donde se descargaran los programas (de repositorios como github). Si se envia vacio o EMPTY se usara el directorio predeterminado 
#         "/var/opt/tools" o "~/tools".
#     08> Ruta base donde se almacena los comandos ("CMD_PATH_BASE/bin"), archivos man1 ("CMD_PATH_BASE/man/man1") y fonts ("CMD_PATH_BASE/share/fonts").
#         Si se envia vacio o EMPTY se usara el directorio predeterminado.
#            > Comandos      : "/usr/local/bin"      (para todos los usuarios) y "~/.local/bin"         (solo para el usuario actual)
#            > Archivos man1 : "/usr/local/man/man1" (para todos los usuarios) y "~/.local/man/man1"    (solo para el usuario actual)
#            > Archivo fuente: "/usr/share/fonts"    (para todos los usuarios) y "~/.local/share/fonts" (solo para el usuario actual)
#     09> Ruta de archivos temporales. Si se envia vacio o EMPTY se usara el directorio predeterminado.
#     10> El estado de la credencial almacenada para el sudo.
#     11> Flag '0' para limpiar el cache de paquetes del sistema operativo. Caso contrario, use 1.
#     12> Actualizar los paquetes del SO. Por defecto es 1 (false), si desea actualizar use 0.
#  > Opción de menu a usar: Configurar el profile del usuario (1) + VIM como Editor 'Crear archivos, Descargar plugins e indexarlos' (4)
USER ${UID}:${GID}
RUN bash ${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash 2 5 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 1 1

#7. Establer las variables de entorno del usuario y cambiar el shell predeterminado 'ash' a 'bash'
ENV PATH="$PATH:/opt/mssql-tools18/bin"
ENTRYPOINT [ "/bin/bash" ]



