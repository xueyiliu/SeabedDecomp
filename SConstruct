import os, sys, re, string
sys.path.append('../../../framework')
import bldutil

progs = '''
ricker
freforrsg
freforrsg_test
decopwaf_Edme
acomuijs2004
pzcomb
elsmuijs2004
elsmuijs2004_pzcali
muijs2002_pzcali
elsmuijs2004_pzcali_test
'''

pyprogs='''
'''

pymods='''
'''

tprogs='''
'''

docs = []

try:  # distributed version
	Import('env root pkgdir libdir bindir')
	env = env.Clone()
except: # local version
	env = bldutil.Debug()
	root = None
	SConscript('../../pwd/SConstruct')

src = Glob('[a-z]*.c')

dynpre = env.get('DYNLIB','')

env.Prepend(CPPPATH=['../../../include'],
	    LIBPATH=['../../../lib'],
	    LIBS=[dynpre+'rsfpwd',
		  dynpre+'rsf', dynpre+'rsfplot'])

for source in src:
	inc = env.RSF_Include(source,prefix='')
	obj = env.StaticObject(source)
	env.Depends(obj,inc)

lapack = env.get('LAPACK')
blas   = env.get('BLAS') and lapack

if blas and not isinstance(lapack,bool):
	env.Prepend(LIBS=lapack)

mains = Split(progs)
for prog in mains:
	sources = ['M' + prog]
	bldutil.depends(env,sources,'M'+prog)	
	if not prog in Split('frt pca epfws orientation coherence fcoh1') or blas:		
		prog = env.Program(prog,map(lambda x: x + '.c',sources))
		if root:
			install = env.Install(bindir,prog)
			
			if dynpre and env['PLATFORM'] == 'darwin':
				env.AddPostAction(install,
						  '%s -change build/api/c/libdrsf.dylib '
						  '%s/libdrsf.dylib %s' % \
					(WhereIs('install_name_tool'),libdir,install[0]))
				env.AddPostAction(install,
						  '%s -change build/user/pwd/libdrsfpwd.dylib '
					'%s/libdrsfpwd.dylib %s' % (WhereIs('install_name_tool'),libdir,install[0]))
				env.AddPostAction(install,
						  '%s -change build/plot/lib/libdrsfplot.dylib '
						  '%s/libdrsfplot.dylib %s' % (WhereIs('install_name_tool'),libdir,install[0]))
				
	else:
		prog = env.RSF_Place('sf'+prog,
				     None,var='LAPACK',package='lapack')
		if root:
			env.Install(bindir,prog)

for prog in Split(tprogs):
	sources = ['T' + prog,prog]
	bldutil.depends(env,sources,prog)
	sources = map(lambda x: x + '.o',sources)
	env.Object('T' + prog + '.c')
	if prog !='pca' or blas:
		env.Program(sources,PROGPREFIX='',PROGSUFFIX='.x')	
	

######################################################################
# SELF-DOCUMENTATION
######################################################################
if root:
	user = os.path.basename(os.getcwd())
	main = 'sf%s.py' % user
	
	docs += map(lambda prog: env.Doc(prog,'M' + prog),mains)  + \
		map(lambda prog: env.Doc(prog,'M'+prog+'.py',lang='python'),pymains)
	env.Depends(docs,'#/framework/rsf/doc.py')	
	doc = env.RSF_Docmerge(main,docs)
	env.Install(pkgdir,doc)
