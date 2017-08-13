project('htsengine', 'c', version: '1.10', license: 'BSD-3')

c_compiler = meson.get_compiler('c')

hts_dependencies = []
pkgconfig_requires = []
pkgconfig_lib_private = []

# Math library
m_dep = c_compiler.find_library('m', required : false)
if m_dep.found()
  hts_dependencies += m_dep
  pkgconfig_lib_private += ['-lm']
endif

hts_defines = []
hts_defines += ['-DSTDC_HEADERS']

hts_embedded = get_option('embedded')
if hts_embedded
  hts_defines += ['-DHTS_EMBEDDED']
endif

hts_festival = get_option('festival')
if hts_festival
  hts_defines += ['-DFESTIVAL']
endif

if host_machine.endian() == 'big'
  hts_defines += ['-DWORDS_BIGENDIAN']
else
  hts_defines += ['-DWORDS_LITTLEENDIAN']
endif

hts_audio = get_option('hts_audio')

if hts_audio == 'auto'
  hts_audio = ['win32', 'portaudio', 'none']
else
  hts_audio = [hts_audio]
endif

host_sys = host_machine.system()
have_selected_audio = false # meson lacks a break instruction
foreach hts_au : hts_audio
  if not have_selected_audio
    if hts_au == 'win32' and host_sys == 'windows' or host_sys == 'wince' or host_sys == 'cygwin' or host_sys == 'mingw'
      winmm_lib = c_compiler.find_library('winmm', required: false)
      if winmm_lib.found()
        have_selected_audio = true
        selected_audio = 'win32'
        hts_dependencies += winmm_lib
        pkgconfig_lib_private += ['-lwinmm']
        hts_defines += ['-DAUDIO_PLAY_WIN32']
      endif
    endif
    if hts_au == 'portaudio'
      portaudio_dep = dependency('portaudio-2.0', version: '>= 19', required: false)
      if portaudio_dep.found()
        have_selected_audio = true
        selected_audio = 'portaudio'
        hts_defines += ['-DAUDIO_PLAY_PORTAUDIO']
        hts_dependencies += portaudio_dep
        pkgconfig_requires += ['portaudio-2.0']
      endif
    endif
    if hts_au == 'none'
      selected_audio = 'none'
      have_selected_audio = true
    endif
  endif
endforeach
message('HTSEngine built with audio: @0@'.format(selected_audio))

sources = [
  'lib/HTS_audio.c',
  'lib/HTS_engine.c',
  'lib/HTS_gstream.c',
  'lib/HTS_hidden.h',
  'lib/HTS_label.c',
  'lib/HTS_misc.c',
  'lib/HTS_model.c',
  'lib/HTS_pstream.c',
  'lib/HTS_sstream.c',
  'lib/HTS_vocoder.c'
]

includes = [include_directories('include')]


libHTSEngine_lib = library('HTSEngine', sources,
  include_directories: includes,
  c_args: hts_defines,
  dependencies: hts_dependencies,
  install: true)

libHTSEngine = declare_dependency(link_with: libHTSEngine_lib,
  include_directories: includes)

hts_engine = executable('hts_engine', ['bin/hts_engine.c'],
  include_directories: includes,
  dependencies: libHTSEngine,
  install: true)

install_headers('include/HTS_engine.h')

########### pkg-config #############

pkg = import('pkgconfig')

pkg.generate(libraries : libHTSEngine_lib,
             name : 'HTSEngine',
             description: 'HTSEngine API',
             requires: pkgconfig_requires,
             libraries_private: pkgconfig_lib_private,
             version: meson.project_version())

