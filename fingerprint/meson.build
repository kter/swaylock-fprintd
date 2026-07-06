gnome = import('gnome')

dbus = dependency('dbus-1')
glib = dependency('glib-2.0', version: '>=2.64.0')
gio_dep = dependency('gio-2.0')

# Fedora's fprintd does not ship the D-Bus introspection XML, so we carry a
# copy extracted from the running daemon (see fingerprint/dbus-xml/).
fprintd_dbus_interfaces = files(
    'dbus-xml/net.reactivated.Fprint.Manager.xml',
    'dbus-xml/net.reactivated.Fprint.Device.xml',
)

fprintd_dbus_sources = gnome.gdbus_codegen('fprintd-dbus',
    sources: fprintd_dbus_interfaces,
    autocleanup: 'all',
    interface_prefix: 'net.reactivated.Fprint.',
    namespace: 'FprintDBus',
    object_manager: true,
)

fingerprint = declare_dependency(
    include_directories: [
        include_directories('..'),
    ],
    sources: [
        fprintd_dbus_sources,
        'fingerprint.c' ,
    ],
    dependencies: [
        gio_dep,
        glib,
        dbus
    ],
)