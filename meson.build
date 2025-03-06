project('vaccel-plugin-template', 'c',
  meson_version : '>=1.1',
  default_options : ['c_std=c11'],
  license : 'Apache-2.0')

plugin_cargs = ['-Wall','-Wextra', '-Wno-unused', '-Werror']

lib_version = meson.project_version().split('-')[0]

libvaccel_dep = dependency('vaccel', version : '>=0.6.1')

subdir('src')
