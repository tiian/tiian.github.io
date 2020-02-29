# FLoM configuration
FLoM uses a cascading configuration principle: every parameter is inspected in 5 different locations and the last occurrence overloads the previous ones.
Every time flom starts, it looks for parameters following this order:

1. hard wired values: they are inside source code and are ever available
2. system default config file: if you didn't change it at *configure time* (command *configure* before command *make*), the standard location would be **/usr/local/etc/flom.conf**; if you changed installation paths, you should determine where **flom.conf** file must be put
3. user default config file: file **.flom** searched inside directory described by environment var **$HOME** or by **/etc/passwd** content if **$HOME** is not set
4. user custom config file: a custom file you specify using "-c" ("\-\-config-file") command line argument
5. specific command line arguments like "-r" ("\-\-resource-name")

System default file, user default config file, user custom config file **must** adhere to the same syntax; an example file is available at **/usr/local/share/doc/flom/flom.conf**.

Using configuration files is not mandatory, you can specify every parameter using command line arguments; many times it's more convenient to store a set of parameters inside a configuration file and use command line arguments for the remaining parameters.
The suggestion is to copy **/usr/local/share/doc/flom/flom.conf** to the desired destination (/usr/local/etc/flom.conf, $HOME/.flom, *my_flom_conf_file*) and customize it as necessary.
