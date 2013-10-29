# -*- python -*-
import os
import os.path
import sys

env = Environment(ENV = {'PATH' : os.environ['PATH']})

vars = Variables(None, ARGUMENTS)
vars.Add(PathVariable('DESTDIR', "Root directory to install in (useful for packaging scripts)", None, PathVariable.PathIsDirCreate))
vars.Add(PathVariable('prefix', "Where to install in the FHS", "/usr/local", PathVariable.PathAccept))

env = Environment(variables=vars)

def calcInstallPath(*elements):
    path = os.path.abspath(os.path.join(*map(env.subst, elements)))
    if 'DESTDIR' in env:
        path = os.path.join(env['DESTDIR'], os.path.relpath(path, start="/"))
    return path

rel_prefix = not os.path.isabs(env['prefix'])
env['prefix'] = os.path.abspath(env['prefix'])
if 'DESTDIR' in env:
    env['DESTDIR'] = os.path.abspath(env['DESTDIR'])
    if rel_prefix:
        print >>sys.stderr, "--!!-- You used a relative prefix with a DESTDIR. This is probably not what you"
        print >>sys.stderr, "--!!-- you want; files will be installed in"
        print >>sys.stderr, "--!!--    %s" % (calcInstallPath("$prefix"),)


env['libpath'] = calcInstallPath("$prefix", "lib")
env['incpath'] = calcInstallPath("$prefix", "include", "hammer")
# TODO: Add pkgconfig

env.MergeFlags("-std=gnu99 -Wall -Wextra -Werror -Wno-unused-parameter -Wno-attributes -lrt")

AddOption("--variant",
          dest="variant",
          nargs=1, type="choice",
          choices=["debug", "opt"],
          default="opt",
          action="store",
          help="Build variant (debug or opt)")

env['BUILDDIR'] = 'build/$VARIANT'

dbg = env.Clone(VARIANT='debug')
dbg.Append(CCFLAGS=['-g'])

opt = env.Clone(VARIANT='opt')
opt.Append(CCFLAGS="-O3")

if GetOption("variant") == 'debug':
    env = dbg
else:
    env = opt

if os.getenv("CC") == "clang":
    env.Replace(CC="clang",
                CXX="clang++")

#rootpath = env['ROOTPATH'] = os.path.abspath('.')
#env.Append(CPPPATH=os.path.join('#', "hammer"))

Export('env')

env.SConscript(["src/SConscript"], variant_dir='build/$VARIANT/src')
env.SConscript(["examples/SConscript"], variant_dir='build/$VARIANT/examples')

env.Command('test', 'build/$VARIANT/src/test_suite', 'env LD_LIBRARY_PATH=build/$VARIANT/src $SOURCE')

env.Alias("install", "$libpath")
env.Alias("install", "$incpath")
