# **Gestión y Auditoría de Permisos en Linux (ES-Ver 🇪🇸)**

## **1. Introducción**
En este documento se detallan los problemas iniciales de permisos en el sistema de archivos de la empresa ficticia **Company_IT** y cómo se han corregido. Se han seguido principios de seguridad y buenas prácticas en administración de sistemas Linux.

## **2. Escenario Inicial**
La estructura de directorios de la empresa era un caos, con permisos mal asignados que ponían en riesgo la seguridad y el flujo de trabajo. A continuación, presentamos el esquema original de permisos y cómo se ha corregido.

---

## **3. Estructura de Directorios y Usuarios**

```
/company_IT/
├── departments/
│   ├── administration/  # Accessible only by the administration group
│   │   ├── contracts.xlxs
│   │   ├── hhrr_data.docx
│   │   └── payrolls.txt
│   ├── external/  # Read-only to prevent modifications
│   │   ├── customer_contracts.docx
│   │   └── supplier_contracts.txt
│   └── guest/  # Permission control for guests
│       ├── guest_env/
│       │   ├── index.html
│       │   ├── script.js
│       │   ├── styles.css
│       │   └── test.py
│       ├── guest_intern.txt
│       ├── jr_access.docx
│       └── sr_access.xlxs
├── projects/
│   ├── completed_projects.md
│   ├── projects_in_progress.py
│   └── version_control.git
└── resources/
    ├── backups.zip
    ├── repositories.git
    └── total_backup_resources.zip
```
### **Usuarios y Grupos** 

- **Usuarios:** `administration`, `jr.`, `sr.` (Cada uno con sus permisos específicos).
- **Grupos:** `guest`, `external`.
- **Roles especiales:** `IT_Admin`, `Admin_Manager` con acceso global a la empresa.

---

## **4. Auditoría de Permisos Inicial**

Para revisar los permisos actuales:


ls -la /company_IT/departments/


Ejemplo de salida problemática:

drwxrwxrwx  5 root root 4096 Mar 14 10:09 administration
-rw-rw-r--  1 user1 user1 2048 Mar 14 09:45 contracts.xlxs

Aquí notamos que *administration* tiene permisos 777, lo que es un grave fallo de seguridad. Debemos corregirlo.

---

## **5. Corrección de Permisos**

Los cambios se aplican **sin usar notación octal**, solo con `chmod` detallado:

### **Administración (Solo acceso interno)**
```
chmod u=rwx,g=---,o=--- /company_IT/departments/administration/
chmod u=rw,g=---,o=--- /company_IT/departments/administration/*
```

Comprobación:
```
ls -la /company_IT/departments/administration/
```
Salida esperada:
```
drwx------  5 administration administration 4096 Mar 14 10:09 administration
-rw-------  1 administration administration 2048 Mar 14 09:45 contracts.xlxs
```

### **External (Solo lectura)**
```
chmod u=rwx,g=r-x,o=r-x /company_IT/departments/external/
chmod u=rw,g=r--,o=r-- /company_IT/departments/external/*
```

### **Guest (Restricciones y Ambiente de Prácticas)**
```
chmod u=rwx,g=rw-,o=--- /company_IT/departments/guest/
chmod u=rwx,g=rwx,o=--- /company_IT/departments/guest/guest_env/
chmod u=rw,g=r--,o=--- /company_IT/departments/guest/guest_intern.txt
```

### **Projects (Accesos por jerarquía)**
```
chmod u=rwx,g=r-x,o=--- /company_IT/projects/
chmod u=rw,g=r--,o=--- /company_IT/projects/completed_projects.md
chmod u=rw,g=rw,o=--- /company_IT/projects/projects_in_progress.py
```

---

## **6. Implementación del Sticky Bit y Protección contra Borrado**

Para evitar que usuarios no autorizados borren archivos:
```
chmod +t /company_IT/resources/
chmod +t /company_IT/projects/
```

Para evitar escritura en archivos críticos:
```
chmod a-w /company_IT/resources/backups.zip
```

Verificación:
```
ls -ld /company_IT/resources/
```
Salida esperada:
```
drwxrwxrwt  5 root root 4096 Mar 14 10:09 resources
```
## **7. Explicación de la cadena de permisos**
La cadena de permisos consta de 10 caracteres:
- **drwxrwxrwx**

| Carácter | Descripción | Afecta |
|----------|-----------------------------------|----------------|
| `d` | Directorio (o `-` para archivo) | Tipo |
| `rwx` | Permisos de usuario (propietario) | Usuario |
| `rwx` | Permisos de grupo | Grupo |
| `rwx` | Permisos de otros | Otros |

- **Permisos de usuario (`rwx`)**:
- **r** (lectura): Permite ver el contenido del archivo.
- **w** (escritura): Permite modificar el archivo o directorio.
- **x** (ejecución): Permite ejecutar el archivo o acceder al directorio.

- **Permisos de grupo (`rwx`)**:
- Igual que los permisos de usuario, pero se aplican al grupo asignado al archivo.

- **Otros permisos (`rwx`)**:
- Igual que los permisos de usuario y grupo, pero se aplican a cualquier otra persona.

--

## **8. Archivos ocultos y permisos**
Los archivos que empiezan por un punto (`.`) se consideran ocultos en Linux, y su administración de permisos funciona igual que la de cualquier otro archivo.

Para modificar los permisos de los archivos ocultos, use el mismo comando `chmod`:

Ejemplo:
- **chmod u=rwx,g=rw-,o=--- /company_IT/departments/.hidden_file.txt**

- Este comando otorga al propietario permisos completos, al grupo acceso de lectura y escritura, y a los demás ningún acceso.

- Puede listar los archivos ocultos con el comando `ls -la`, que mostrará todos los archivos, incluidos los ocultos.

--

## **8. Resumen**

Se han corregido los permisos para garantizar:
- Seguridad en archivos sensibles (Administración, Recursos).
- Acceso controlado a invitados y externos.
- Jerarquización en `projects`.
- Protección contra eliminación accidental.

El sistema ahora sigue el principio de **mínimos privilegios**, asegurando integridad y funcionalidad.

-- 


