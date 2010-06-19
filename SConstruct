# vim: ft=python expandtab

import os
from site_init import *
from xml.etree.ElementTree import Element, SubElement, XMLTreeBuilder, tostring
from xml.dom import minidom
from uuid import uuid4

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

env['PDB'] = libname + '.pdb'

env.DotIn('scripts/' + name + '.pc', 'scripts/libpng.pc.in')

libpng_SOURCES = Split("png.c pngset.c pngget.c pngrutil.c pngtrans.c pngwutil.c \
	pngread.c pngrio.c pngwio.c pngwrite.c pngrtran.c \
	pngwtran.c pngmem.c pngerror.c pngpread.c")

env.RES('scripts/pngw32.res', 'scripts/pngw32.rc')
dllname = libname + env['LIB_SUFFIX'] + '.dll'
dll = env.SharedLibrary([dllname, name + '.lib'], libpng_SOURCES + ['scripts/pngw32.def', 'scripts/pngw32.res'])

env.AddPostAction(dll, 'mt.exe -nologo -manifest ${TARGET}.manifest -outputresource:$TARGET;2')

dev_root = Element("Wix", xmlns='http://schemas.microsoft.com/wix/2006/wi')
dev_m = SubElement(dev_root, 'Module', Id='LibpngDev', Language='1033', Version=PNG_VERSION_STRING)
dev_p = SubElement(dev_m, 'Package', Id=str(uuid4()).upper(), Description='Libpng devlopement package',
                Comments='This is a windows installer for libpng library devlopment files',
                Manufacturer='Gnome4Win', InstallerVersion='200')
dev_target = SubElement(dev_m, 'Directory', Id='TARGETDIR', Name='SourceDir')

run_root = Element("Wix", xmlns='http://schemas.microsoft.com/wix/2006/wi')
run_m = SubElement(run_root, 'Module', Id='LibpngRun', Language='1033', Version=PNG_VERSION_STRING)
run_p = SubElement(run_m, 'Package', Id=str(uuid4()).upper(), Description='Libpng runtime package',
                Comments='This is a windows installer for libpng library runtime files',
                Manufacturer='Gnome4Win', InstallerVersion='200')
run_target = SubElement(run_m, 'Directory', Id='TARGETDIR', Name='SourceDir')

env.Alias('install', env.Install('$PREFIX/include/' + libname, ['png.h', 'pngconf.h']))
dev_dinclude = SubElement(dev_target, 'Directory', Id='include', Name='include')
dev_cheader = SubElement(dev_dinclude, "Component", Id='headers', Guid=str(uuid4()).upper())
FileElement(dev_cheader, ['png.h', 'pngconf.h'], 'include/' + libname, env)

env.Alias('install', env.Install('$PREFIX/bin', libname + env['LIB_SUFFIX'] + '.dll'))
run_bin = SubElement(run_target, 'Directory', Id='bin', Name='bin')
run_cdll = SubElement(run_bin, "Component", Id='dlls', Guid=str(uuid4()).upper())
FileElement(run_cdll, dllname, 'bin', env)

env.Alias('install', env.Install('$PREFIX/lib', name + '.lib'))
dev_lib = SubElement(dev_target, 'Directory', Id='lib', Name='lib')
dev_clib = SubElement(dev_lib, "Component", Id='libs', Guid=str(uuid4()).upper())
FileElement(dev_clib, name+'.lib', 'lib', env)

env.Alias('install', env.Install('$PREFIX/lib/pkgconfig', 'scripts/' + name + '.pc'))
dev_pkgconfig = SubElement(dev_lib, 'Directory', Id='pkgconfig', Name='pkgconfig')
dev_cpkgconfig = SubElement(dev_pkgconfig, "Component", Id='pkgconfigs', Guid=str(uuid4()).upper())
FileElement(dev_cpkgconfig, name+'.pc', 'lib/pkgconfig', env)

if env['DEBUG']:
    env.Alias('install', env.Install('$PREFIX/pdb', env['PDB']))
    dev_pdb = SubElement(dev_target, 'Directory', Id='pdb', Name='pdb')
    dev_cpdb = SubElement(dev_pdb, "Component", Id='pdbs', Guid=str(uuid4()).upper())
    FileElement(dev_cpdb, env['PDB'], 'pdb', env)

env_dev = env.Clone(XML=dev_root)
env_dev.Command('pngdev.wxs', [], generate_wxs)

env_run = env.Clone(XML=run_root)
env_run.Command('pngrun.wxs', [], generate_wxs)

env.Depends(['pngrun.wxs', 'pngdev.wxs'], 'SConstruct')
env.Alias('install', env.Install('$PREFIX/wxs', ['pngrun.wxs', 'pngdev.wxs']))
