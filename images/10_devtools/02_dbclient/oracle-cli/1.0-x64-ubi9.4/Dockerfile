#
#Limpieza:
#  podman image rm lucianoepc/oracle-cli:1.0-x64-ubi9.4
#  podman image prune 
#
#Ejemplo del comando de creación/compilación de la imagen:
#  cd /home/lucianoepc/code/own1/files/imagebuilding
#  podman build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/oracle-cli:1.0-x64-ubi9.4 ./containerfiles/10_devtools/02_dbclient/oracle-cli/1.0-x64-ubi9.4/
#  podman run -it --rm --name test01 lucianoepc/oracle-cli:1.0-x64-ubi9.4
#  $ sqlplus -V


#-------------------------------------------------------------------------------------
# Layer intermedia
#-------------------------------------------------------------------------------------
FROM registry.access.redhat.com/ubi9/ubi:9.4 as builder

#1. Constantes
ARG PCKG_BASE_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-basic-linux.x64-21.13.0.0.0dbru.zip'
ARG PCKG_SQLPLUS_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-sqlplus-linux.x64-21.13.0.0.0dbru.zip'
ARG PCKG_TOOLS_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-tools-linux.x64-21.13.0.0.0dbru.zip'
ARG PCKG_SDK_INSTALL=0
ARG PCKG_SDK_URL='https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-sdk-linux.x64-21.13.0.0.0dbru.zip'

#2. Instalar programas basicos y archivos del profile del usuario.
#   Ya cuenta con 'curl-minimal', se genera conflictos instalar el paquete completo 'curl'.
#RUN dnf -y install git curl unzip \
RUN dnf -y install git unzip \
    && git clone https://github.com/lestebanpc/dotfiles.git ~/.files

#3. Descargar y organizar los archivos de 'Oracle Instant Client' v21.13
RUN mkdir -m 777 /cmd \
    && mkdir -pm 755 /oracle/db/client \
    && mkdir -m 755 /oracle/db/client/libext/ \
    && mkdir -m 755 /oracle/db/client/bin/ \
    && curl -fLo package-basic.zip ${PCKG_BASE_URL} \
    && unzip package-basic.zip -d /oracle/db/client \
    && curl -fLo package-sqlplus.zip ${PCKG_SQLPLUS_URL} \
    && unzip package-sqlplus.zip -d /oracle/db/client \
    && curl -fLo package-tools.zip ${PCKG_TOOLS_URL} \
    && unzip package-tools.zip -d /oracle/db/client \
    && if [ ${PCKG_SDK_INSTALL} -eq 0 ]; then \
        curl -fLo package-sdk.zip ${PCKG_SDK_URL}; \
        unzip package-sdk.zip -d /oracle/db/client; \
        rm package-sdk.zip; \
    fi \
    && rm package-basic.zip package-sqlplus.zip package-tools.zip \
    && mv /oracle/db/client/instantclient_21_13 /oracle/db/client/lib/ \
    && if [ ${PCKG_SDK_INSTALL} -eq 0 ]; then \
        mv /oracle/db/client/lib/sdk/  /oracle/db/client/; \
    fi \
    && mv /oracle/db/client/lib/*_LICENSE   /oracle/db/client/ \
    && mv /oracle/db/client/lib/*_README    /oracle/db/client/ \
    && mv /oracle/db/client/lib/network/    /oracle/db/client/ \ 
    && mv /oracle/db/client/lib/*.jar       /oracle/db/client/libext/ \
    && mv /oracle/db/client/lib/adrci       /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/genezi      /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/glogin.sql  /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/sqlplus     /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/uidrvci     /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/exp         /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/expdp       /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/imp         /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/impdp       /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/sqlldr      /oracle/db/client/bin/ \
    && mv /oracle/db/client/lib/wrc         /oracle/db/client/bin/ 


#4. Descargar el VIM de plugins y comandos indicando en parametro.
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
#     01> Tipo de invocación (sin menu): 1/2 (sin usar un menu interactivo/no-interactivo)
#     02> Opciones de menu a ejecutar: entero positivo.
#     03> ID de los repositorios de comandos a configurar, separados por coma. Si no desea configurarse ninguno envie "EMPTY".
#     04> ID de los paquetes del repositorio del SO, separados por coma, a instalar si elige la opcion de menu 32. Si desea usar el los los paquetes por defecto envie "EMPTY".
#         Los paquetes por defecto que son: Curl, UnZip, OpenSSL y Tmux.
#     05> Ruta donde se descargaran los programas (de repositorios como github). Si se envia vacio o EMPTY se usara el directorio predeterminado "/opt/tools" o "~/tools".
#     06> Ruta base donde se almacena los comandos ("CMD_PATH_BASE/bin"), archivos man1 ("CMD_PATH_BASE/man/man1") y fonts ("CMD_PATH_BASE/share/fonts").
#         Si se envia vacio o EMPTY se usara el directorio predeterminado.
#            > Comandos      : "/usr/local/bin"      (para todos los usuarios) y "~/.local/bin"         (solo para el usuario actual)
#            > Archivos man1 : "/usr/local/man/man1" (para todos los usuarios) y "~/.local/man/man1"    (solo para el usuario actual)
#            > Archivo fuente: "/usr/share/fonts"    (para todos los usuarios) y "~/.local/share/fonts" (solo para el usuario actual)
#     07> Ruta de archivos temporales. Si se envia vacio o EMPTY se usara el directorio predeterminado.
#     08> El estado de la credencial almacenada para el sudo.
#     09> Flag '0' para limpiar el cache de paquetes del sistema operativo. Caso contrario, use 1.
#     10> Actualizar los paquetes del SO. Por defecto es 1 (false), si desea actualizar use 0.
#     11> El GID y UID del usuario que ejecuta el script, siempre que no se el owner de repositorio, en formato "UID:GID"
#  > Opción de menu a usar: (131072) Descargar plugins de Editor de VIM
RUN bash ~/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash 2 131072 'jq,yq,fd,fzf,bat,ripgrep,oh-my-posh,eza,zoxide' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' '/cmd' 'EMPTY' -1 1 1


#-------------------------------------------------------------------------------------
# Layer final
#-------------------------------------------------------------------------------------
FROM registry.access.redhat.com/ubi9/ubi:9.4 as final

#1. Parametros de entrada
ARG UID
ARG GID


#2. Constantes
ARG USER_NAME=appuser
ARG HOME_PATH=/home/appuser

#3. Crear el usuario/Grupo para la aplicación
RUN dnf -y install libaio \
    #sudo curl unzip \
    && dnf -y clean all \
    && groupadd -g $GID ${USER_NAME} \
    && useradd -u $UID -g ${USER_NAME} -d ${HOME_PATH} -s /bin/bash ${USER_NAME} \
    && mkdir -p ${HOME_PATH} \
    && chown $UID:$GID ${HOME_PATH} \
    #&& echo "${USER_NAME} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \
    && mkdir -pm 755 /var/opt/tools/oracle \
    && chown -R $UID:$GID /var/opt/tools/ \ 
    && mkdir -m 755 ${HOME_PATH}/.vim/ \
    && chown $UID:$GID ${HOME_PATH}/.vim/ \
    && mkdir -m 755 ${HOME_PATH}/.files/ \
    && chown $UID:$GID ${HOME_PATH}/.files/

#4. Copiar archivos de usuario del profile y oracle client
COPY --from=builder /cmd/bin/ /usr/local/bin
COPY --from=builder /cmd/man/man1 /usr/local/man/man1
COPY --chown=${UID}:${GID} --from=builder /oracle/ /var/opt/tools/oracle
COPY --chown=${UID}:${GID} --from=builder /root/.files/ ${HOME_PATH}/.files/
COPY --chown=${UID}:${GID} --from=builder /root/.vim/ ${HOME_PATH}/.vim/

#5. Instalar libreria compartidas e instalar VIM
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
#  > Opción de menu a usar: Instalar VIM (32768)
RUN echo /var/opt/tools/oracle/db/client/lib > /etc/ld.so.conf.d/oracle-instantclient.conf \
    && ldconfig \
    && bash "${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash" 2 32768 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 0 1 "${UID}:${GID}"

#6. Configurar el profile como Editor:
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
#  > Opción de menu a usar:  1 (User profile) + 64 (Crear archivos de configuración y Indexar plugins)
USER ${UID}:${GID}
RUN bash "${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash" 2 65 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 1 1

#Variables de entorno predefinidos
ENV PATH="$PATH:/var/opt/tools/oracle/db/client/bin"


