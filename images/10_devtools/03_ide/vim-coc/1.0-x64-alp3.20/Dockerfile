#
#Limpieza:
#  podman image rm lucianoepc/vim-coc:1.0-x64-alp3.20
#  podman image prune 
#
#Ejemplo del comando de creación/compilación de la imagen:
#  cd /home/lucianoepc/code/own1/files/imagebuilding
#  podman build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/vim-coc:1.0-x64-alp3.20 ./containerfiles/10_devtools/03_ide/vim-coc/1.0-x64-alp3.20/
#  podman run -it --rm --name test01 lucianoepc/vim-coc:1.0-x64-alp3.20
#

#-------------------------------------------------------------------------------------
# Layer final
#-------------------------------------------------------------------------------------
FROM alpine:3.20

#Parametros de entrada
ARG UID
ARG GID

#Constantes
ARG USER_NAME=appuser
ARG HOME_PATH=/home/appuser

#Crear el usuario/Grupo para desarrollo e instalar git/sudo
#    > 'libstdc++' se usa para NodeJS en Alpine/musl
RUN apk --no-cache add bash git libstdc++ sudo openssh-client \
    && addgroup -g $GID ${USER_NAME} \
    && adduser -D -u $UID -G ${USER_NAME} -h ${HOME_PATH} -s /bin/bash ${USER_NAME} \
    && mkdir -p ${HOME_PATH} \
    && chown $UID:$GID ${HOME_PATH} \
    && mkdir -m 755 /opt/tools/ \
    && chown $UID:$GID /opt/tools/ \ 
    && echo "${USER_NAME} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


#4. Descargar archivos de usuario
USER ${UID}:${GID}
RUN git clone https://github.com/lestebanpc/dotfiles.git ${HOME_PATH}/.files

#5. Instalar paquetes basicos, comandos basicos, NodeJS/Python y VIM:
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
#   > Opción de menu a usar: Paquetes basicos (1024) + Instalar NodeJS (4096) + Instalar Python (16384) + Instalar paquetes globales de NodeJS (8192) + Instalar VIM (32768)
USER 0:0
RUN bash "${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash" 2 62464 'jq,yq,fd,bat,delta,ripgrep,oh-my-posh,eza,zoxide,kubectl,oc' \
         'curl,unzip,openssl,tmux,fzf' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 0 1

#6. Configurar VIM como IDE y configurar el profile como Developer:
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
#   > Opción de menu a usar: User profile (1) + VIM como IDE (8) + Instalar paquetes de usuario de Python (2)
USER ${UID}:${GID}
RUN bash "${HOME_PATH}/.files/shell/bash/bin/linuxsetup/00_setup_summary.bash" 2 11 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' 'EMPTY' -1 1 1



#Cambiar el shell predeterminado 'ash' a 'bash'
ENTRYPOINT [ "/bin/bash" ]



