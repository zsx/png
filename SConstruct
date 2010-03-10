# vim: ft=python expandtab

import os
from site_init import GBuilder, Initialize

opts = Variables()
opts.Add(PathVariable('PREFIX', 'Installation prefix', os.path.expanduser('~/FOSS'), PathVariable.PathIsDirCreate))
opts.Add(BoolVariable('DEBUG', 'Build with Debugging information', 0))
opts.Add(PathVariable('PERL', 'Path to the executable perl', r'C:\Perl\bin\perl.exe', PathVariable.PathIsFile))
opts.Add(BoolVariable('WITH_OSMSVCRT', 'Link with the os supplied msvcrt.dll instead of the one supplied by the compiler (msvcr90.dll, for instance)', 0))

env = Environment(variables = opts,
                  ENV=os.environ, tools = ['default', GBuilder])

Initialize(env)

PNG_VERSION_MAJOR=1
PNG_VERSION_MINOR=2
PNG_VERSION_RELEASE=40
PNG_VERSION_STRING="%d.%d.%d" % (PNG_VERSION_MAJOR, PNG_VERSION_MINOR, PNG_VERSION_RELEASE)

env['DOT_IN_SUBS'] = {'@PACKAGE_VERSION@': PNG_VERSION_STRING,
					  '@VERSION@': PNG_VERSION_STRING,
					  '@PNG_VERSION_MAJOR@': str(PNG_VERSION_MAJOR),
					  '@PNG_VERSION_MINOR@': str(PNG_VERSION_MINOR),
					  '@PNG_VERSION_RELEASE@': str(PNG_VERSION_RELEASE),
					  '@prefix@': env['PREFIX'],
 					  '@exec_prefix@': '${prefix}/bin',
					  '@libdir@': '${prefix}/lib',
					  '@includedir@': '${prefix}/include'}

env.AppendENVPath('PKGCONFIG_PATH', '$PREFIX/lib/pkgconfig')
env.ParseConfig('pkg-config zlib --cflags --libs')
env.Append(CPPPATH=['#'])
env.Append(CFLAGS=env['DEBUG_CFLAGS'])
env.Append(CPPDEFINES=env['DEBUG_CPPDEFINES'])

if env['WITH_OSMSVCRT']:
    env['LIB_SUFFIX'] = '-0'

name = 'png%s%s' % (PNG_VERSION_MAJOR, PNG_VERSION_MINOR)
libname = 'lib' + name

env.DotIn('scripts/' + name + '.pc', 'scripts/libpng.pc.in')
env.Alias('install', env.Install('$PREFIX/lib/pkgconfig', 'scripts/' + name + '.pc'))

libpng_SOURCES = Split("png.c pngset.c pngget.c pngrutil.c pngtrans.c pngwutil.c \
	pngread.c pngrio.c pngwio.c pngwrite.c pngrtran.c \
	pngwtran.c pngmem.c pngerror.c pngpread.c")

env.RES('scripts/pngw32.res', 'scripts/pngw32.rc')
dll = env.SharedLibrary([libname + env['LIB_SUFFIX'] + '.dll', name + '.lib'], libpng_SOURCES + ['scripts/pngw32.def', 'scripts/pngw32.res'])

env.AddPostAction(dll, 'mt.exe -nologo -manifest ${TARGET}.manifest -outputresource:$TARGET;2')

env.Alias('install', env.Install('$PREFIX/include/' + libname, ['png.h', 'pngconf.h']))
env.Alias('install', env.Install('$PREFIX/bin', libname + env['LIB_SUFFIX'] + '.dll'))
env.Alias('install', env.Install('$PREFIX/lib', name + '.lib'))
