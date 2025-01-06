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
FROM registry.access.redhat.com/ubi8/ubi:8.9

#1. Parametros de entrada
ARG UID
ARG GID

#2. Constantes
ARG USER_NAME=appuser
ARG HOME_PATH=/home/appuser
ARG PCKG_BASE_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-basic-linux.x64-21.13.0.0.0dbru.zip'
ARG PCKG_SQLPLUS_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-sqlplus-linux.x64-21.13.0.0.0dbru.zip'
ARG PCKG_TOOLS_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-tools-linux.x64-21.13.0.0.0dbru.zip'
ARG PCKG_SDK_INSTALL=0
ARG PCKG_SDK_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-sdk-linux.x64-21.13.0.0.0dbru.zip'

#2. Instalar programas basicos y crear el usuario/grupo para la aplicación
RUN dnf -y install git sudo curl unzip libaio \
    && dnf -y clean all \
    && groupadd -g $GID ${USER_NAME} \
    && useradd -u $UID -g ${USER_NAME} -d ${HOME_PATH} -s /bin/bash ${USER_NAME} \
    && mkdir -p ${HOME_PATH} \
    && chown $UID:$GID ${HOME_PATH} 
    #&& echo "${USER_NAME} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \

#3. Instalar el 'Oracle Instant Client' v21.13
RUN mkdir -pm 755 /var/opt/tools/oracle/db/client \
    && mkdir -m 755 /var/opt/tools/oracle/db/client/libext/ \
    && mkdir -m 755 /var/opt/tools/oracle/db/client/bin/ \
    && curl -fLo package-basic.zip ${PCKG_BASE_URL} \
    && unzip package-basic.zip -d /var/opt/tools/oracle/db/client \
    && curl -fLo package-sqlplus.zip ${PCKG_SQLPLUS_URL} \
    && unzip package-sqlplus.zip -d /var/opt/tools/oracle/db/client \
    && curl -fLo package-tools.zip ${PCKG_TOOLS_URL} \
    && unzip package-tools.zip -d /var/opt/tools/oracle/db/client \
    && if [ ${PCKG_SDK_INSTALL} -eq 0 ]; then \
        curl -fLo package-sdk.zip ${PCKG_SDK_URL}; \
        unzip package-sdk.zip -d /var/opt/tools/oracle/db/client; \
        rm package-sdk.zip; \
    fi \
    && rm package-basic.zip package-sqlplus.zip package-tools.zip \
    && mv /var/opt/tools/oracle/db/client/instantclient_21_13 /var/opt/tools/oracle/db/client/lib/ \
    && if [ ${PCKG_SDK_INSTALL} -eq 0 ]; then \
        mv /var/opt/tools/oracle/db/client/lib/sdk/  /var/opt/tools/oracle/db/client/; \
    fi \
    && mv /var/opt/tools/oracle/db/client/lib/*_LICENSE   /var/opt/tools/oracle/db/client/ \
    && mv /var/opt/tools/oracle/db/client/lib/*_README    /var/opt/tools/oracle/db/client/ \
    && mv /var/opt/tools/oracle/db/client/lib/network/    /var/opt/tools/oracle/db/client/ \ 
    && chown -R $UID:$GID /var/opt/tools/oracle/db/client/network/ \ 
    && mv /var/opt/tools/oracle/db/client/lib/*.jar       /var/opt/tools/oracle/db/client/libext/ \
    && mv /var/opt/tools/oracle/db/client/lib/adrci       /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/genezi      /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/glogin.sql  /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/sqlplus     /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/uidrvci     /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/exp         /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/expdp       /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/imp         /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/impdp       /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/sqlldr      /var/opt/tools/oracle/db/client/bin/ \
    && mv /var/opt/tools/oracle/db/client/lib/wrc         /var/opt/tools/oracle/db/client/bin/ \
    && echo /var/opt/tools/oracle/db/client/lib > /etc/ld.so.conf.d/oracle-instantclient.conf \
    && ldconfig

#3. Descargar archivos del usuario
USER ${UID}:${GID}
RUN git clone https://github.com/lestebanpc/dotfiles.git ${HOME_PATH}/.files

#4. Instalar paquetes basicos adicionales (openssl y VIM) y comandos basicos
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
#  > Opción de menu a usar: Paquetes basicos 'openssl' (1024) + VIM (32768) 
USER 0:0
RUN bash ${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash 2 33792 'jq,yq,fd,fzf,bat,ripgrep,oh-my-posh,eza,zoxide' 'openssl' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 0 1

#5. Instalar el profile como Editor
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

#6. Establecer el PATH los comandos del Oracle Client
ENV PATH="$PATH:/var/opt/tools/oracle/db/client/bin"


