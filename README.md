<div align="center">
<table>
    <theader>
        <tr>
            <th><img src="https://github.com/rescobedoulasalle/git_github/blob/main/ulasalle.png?raw=true" alt="EPIS" style="width:50%; height:auto"/></th>
            <th>
                <span style="font-weight:bold;">UNIVERSIDAD LA SALLE</span><br />
                <span style="font-weight:bold;">FACULTAD DE INGENIERÍAS</span><br />
                <span style="font-weight:bold;">DEPARTAMENTO DE INGENIERÍA Y MATEMÁTICAS</span><br />
                <span style="font-weight:bold;">CARRERA PROFESIONAL DE INGENIERÍA DE SOFTWARE</span>
            </th>            
        </tr>
    </theader>
    
</table>
</div>

<div align="center">
<span style="font-weight:bold;">GUÍA DE LABORATORIO</span><br />
</div>

<table>
    <theader>
        <tr><th colspan="2">INFORMACIÓN BÁSICA</th></tr>
    </theader>
<tbody>

<tr><td>TÍTULO DE LA PRÁCTICA:</td><td>Kubernetes</td></tr>
<tr><td colspan="2">RECURSOS:
    <ul>
        <li>https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/</li>
        <li>https://kubernetes.io/docs/concepts/storage/persistent-volumes/</li>
        <li>https://kubernetes.io/blog/2021/09/13/read-write-once-pod-access-mode-alpha/</li>
        <li>http://labs.play-with-k8s.com/</li>
        <li>https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk</li>
        <li>https://kubernetes.io/docs/concepts/configuration/secret/</li>
        <li>https://stackoverflow.com/questions/49260135/unable-to-connect-to-the-server-dial-tcp-i-o-time-out</li>
    </ul>
</td>
</<tr>
<tr><td colspan="2">DOCENTES:
    <ul>
        <li>Richart Smith Escobedo Quispe  - r.escobedo@ulasalle.edu.pe</li>
    </ul>
</td>
</<tr>
</tdbody>
</table>

# Kubernetes

[![License][license]][license-file]
[![Downloads][downloads]][releases]
[![Last Commit][last-commit]][releases]

[![Debian][Debian]][debian-site]
[![Git][Git]][git-site]
[![GitHub][GitHub]][github-site]
[![Vim][Vim]][vim-site]
[![Java][Java]][java-site]

#

## OBJETIVOS Y TEMAS

### OBJETIVOS
-   Implementar un sitio de WordPress y una base de datos MySQL usando Minikube.
-   Crear Volúmenes persistentes.
-   Personalizar las configuraciones.
-   Trabajar en un directorio de personalización.
-   Limpiar.

### TEMAS
- Volúmenes persistentes.
- Docker vs VMs
- Arquitectura de Docker
- Comandos en Docker

## CONTENIDO DE LA GUÍA

### MARCO CONCEPTUAL

#### Vólumenes persistentes : Gestión del almacenamiento

-   PersistentVolume (PV)
    -   Es una pieza de almacenamiento en el clúster que un administrador ha aprovisionado manualmente o que Kubernetes ha aprovisionado dinámicamente mediante StorageClass.

-   PersistentVolumeClaim (PVC) 
    -   Es una solicitud de almacenamiento por parte de un usuario que puede ser cumplida por un PV.

-   PersistentVolumes y PersistentVolumeClaims son independientes de los ciclos de vida de los pods y preservan los datos mediante el reinicio, la reprogramación e incluso la eliminación de pods.

## EJERCICIO RESUELTO POR EL DOCENTE
-   kubernetes 1.9 y posteriores

-   Tener un clúster de Kubernetes y la herramienta de línea de comandos kubectl debe estar configurada para comunicarse con su clúster.

-   Entorno online: http://labs.play-with-k8s.com/

-   Verificar la versión
    ```sh
    WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
    Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.2", GitCommit:"f66044f4361b9f1f96f0053dd46cb7dce5e990a8", GitTreeState:"clean", BuildDate:"2022-06-15T14:22:29Z", GoVersion:"go1.18.3", Compiler:"gc", Platform:"linux/amd64"}
    Kustomize Version: v4.5.4
    Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:19:12Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/amd64"}
    ```

-   Ejemplo probado con versiones 1.14 o superiores

-   Archivos de configuraciones:
    -   mysql-deployment.yaml
    -   wordpress-deployment.yaml

-   MySQL y Wordpress requieren un PersistentVolume para almacenar datos.
-   Sus PersistentVolumeClaims se crearán en el paso de implementación.

-   Muchos entornos de clúster tienen instalado un StorageClass predeterminado. Cuando no se especifica un StorageClass en PersistentVolumeClaim, se usa el StorageClass predeterminado del clúster en su lugar.

-   Cuando se crea un PersistentVolumeClaim, un PersistentVolume se aprovisiona dinámicamente en función de la configuración de StorageClass.

-   Secretos:    
    -   Objetos que almacenan una parte de los datos confidenciales, como una contraseña o clave.
    -   Desde 1.14, kubectladmite la gestión de objetos de Kubernetes mediante un archivo de personalización. 
    -   Puedes crear un Secreto por generadores en kustomization.yaml.
    -   Reemplazar YOUR_PASSWORDcon la contraseña que desea utilizar.
    -   ```sh
        cat <<EOF >./kustomization.yaml
        secretGenerator:
        - name: mysql-pass
        literals:
        - password=YOUR_PASSWORD
        EOF
        ```
-   Manifiesto que describe una implementación de MySQL de instancia única.
-   El contenedor de MySQL monta PersistentVolume en /var/lib/mysql. La MYSQL_ROOT_PASSWORDvariable de entorno establece la contraseña de la base de datos del secreto.
    -   mysql-deployment.yaml

-   Manifiesto describe una implementación de WordPress de instancia única.
-   El contenedor de WordPress monta PersistentVolume en /var/www/htmllos archivos de datos del sitio web. 
-   La WORDPRESS_DB_HOSTvariable de entorno establece el nombre del Servicio MySQL definido anteriormente, y WordPress accederá a la base de datos por Servicio. 
-   La WORDPRESS_DB_PASSWORDvariable de entorno establece la contraseña de la base de datos a partir del Secret kustomize generado.
    -   wordpress-deployment.yaml

-   Agréguelos al kustomization.yamlarchivo.
    ```sh
    cat <<EOF >>./kustomization.yaml
    resources:
    - mysql-deployment.yaml
    - wordpress-deployment.yaml
    EOF
    ```

-   Aplique y verifique
    ```sh
    kubectl apply -k ./
    ```

-   Verificar que todos los objetos existen
    ```sh
    kubectl get secrets
    ```
    ```sh
    NAME                    TYPE                                  DATA   AGE
    mysql-pass-c57bb4t7mf   Opaque                                1      9s
    ```

-   Verifique que un PersistentVolume se haya aprovisionado dinámicamente
    ```sh
    kubectl get pvc
    ```
    ```sh
    NAME             STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
    mysql-pv-claim   Bound     pvc-8cbd7b2e-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
    wp-pv-claim      Bound     pvc-8cd0df54-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
    ```

-   Verifique que el Pod se esté ejecutando ejecutando el siguiente comando
    ```sh
    kubectl get pods
    ```
    ```sh
    NAME                               READY     STATUS    RESTARTS   AGE
    wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s
    ```

-   Verifique que el servicio se esté ejecutando ejecutando el siguiente comando:
    ```sh
    kubectl get services wordpress
    ```
    ```sh
    NAME        TYPE            CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
    wordpress   LoadBalancer    10.0.0.89    <pending>     80:32406/TCP   4m
    ```

-   Obtener la dirección IP del servicio de WordPress
    ```sh
    minikube service wordpress --url
    ```
    ```sh
    http://1.2.3.4:32406
    ```

-    Eliminar su Secreto, Implementaciones, Servicios y PersistentVolumeClaims
    ```sh
    kubectl delete -k ./
    ```

#

## EJERCICIOS PROPUESTOS

-   1. Usar WordPress Helm Chart para implementar WordPress en producción.

#

## CUESTIONARIO

-   ¿Qué tipos de volúmenes persistentes exixten?
-   ¿Qué modos de acceso se le pueden dar a los volúmenes persistentes?
-   

#

## REFERENCIAS
    -   https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/
    -   https://kubernetes.io/docs/concepts/storage/persistent-volumes/
    -   https://kubernetes.io/blog/2021/09/13/read-write-once-pod-access-mode-alpha/
    -   http://labs.play-with-k8s.com/
    -   https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk
    -   https://kubernetes.io/docs/concepts/configuration/secret/
    -   https://stackoverflow.com/questions/49260135/unable-to-connect-to-the-server-dial-tcp-i-o-time-out

#

[license]: https://img.shields.io/github/license/rescobedoulasalle/git_github?label=rescobedoulasalle
[license-file]: https://github.com/rescobedoulasalle/git_github/blob/main/LICENSE

[downloads]: https://img.shields.io/github/downloads/rescobedoulasalle/git_github/total?label=Downloads
[releases]: https://github.com/rescobedoulasalle/git_github/releases/

[last-commit]: https://img.shields.io/github/last-commit/rescobedoulasalle/git_github?label=Last%20Commit

[Debian]: https://img.shields.io/badge/Debian-D70A53?style=for-the-badge&logo=debian&logoColor=white
[debian-site]: https://www.debian.org/index.es.html

[Git]: https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white
[git-site]: https://git-scm.com/

[GitHub]: https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white
[github-site]: https://github.com/

[Vim]: https://img.shields.io/badge/VIM-%2311AB00.svg?style=for-the-badge&logo=vim&logoColor=white
[vim-site]: https://www.vim.org/




[![Debian][Debian]][debian-site]
[![Git][Git]][git-site]
[![GitHub][GitHub]][github-site]
[![Vim][Vim]][vim-site]
[![Java][Java]][java-site]

[![License][license]][license-file]
[![Downloads][downloads]][releases]
[![Last Commit][last-commit]][releases]

