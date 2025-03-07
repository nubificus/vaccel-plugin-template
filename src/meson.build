plugin_headers = files([
])

plugin_sources = files([
  'vaccel.c',
])

plugin_sources += [plugin_sources, plugin_headers]
plugin_includes = include_directories('.')

plugin_deps = [
  libvaccel_dep,
]

libvaccel_template = library('vaccel-template',
  plugin_sources,
  c_args : plugin_cargs,
  include_directories : plugin_includes,
  dependencies : plugin_deps,
  install : true)

pkg_mod = import('pkgconfig')
pkg_mod.generate(libvaccel_template,
  name : 'vaccel-template',
  description : 'A template vAccel plugin')
