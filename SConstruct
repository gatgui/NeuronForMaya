import os
import sys
import shutil
import excons
import excons.tools.maya as maya

SDKURL = "https://neuronmocap.com/system/files/software/NeuronDataReader%20SDK%20b17_0.zip"
SDKDIR = "NeuronSDK"
SDKARC = "NeuronSDKb17.zip"

if sys.platform not in ("win32", "darwin"):
   print("Unsupported platform")
   sys.exit(1)

maya.SetupMscver()

env = excons.MakeBaseEnv()

if not os.path.isdir(SDKDIR):
   if not os.path.isfile(SDKARC):
      print("Dowloading NeuroSDK ...")
      try:
         import urllib2
         url = urllib2.urlopen(SDKURL)
         with open(SDKARC, "wb") as outf:
            outf.write(url.read())
      except Exception, e:
         print("Cannot download Neuron SDK b17 (%s).\nTry to download it manually from https://neuronmocap.com and extract in a folder named '%s'" % (e, SDKDIR))
         sys.exit(1)

   if os.path.isfile(SDKARC):
      print("Extracting NeuroSDK ...")
      dl = set(os.listdir("."))
      try:
         import zipfile
         with zipfile.ZipFile(SDKARC) as zf:
            zf.extractall(".")
      except Exception, e:
         print("Failed to extract SDK archive '%s' (%s)." % (SDKARC, e))
         sys.exit(1)
      ndl = set(os.listdir("."))
      ddl = list(ndl.difference(dl))
      if len(ddl) == 1 and ddl[0].startswith("NeuronDataReader"):
         print("Rename %s -> %s" % (ddl[0], SDKDIR))
         shutil.move(ddl[0], SDKDIR)
      else:
         print("Unexpected archive content")
         sys.exit(1)

WINDLL = SDKDIR + "/lib/win/desktop/x64/NeuronDataReader.dll"
MACLIB = SDKDIR + "/lib/mac/libNeuronDataReader.dylib"
RUNTIME = (WINDLL if sys.platform == "win32" else MACLIB)
if not os.path.isfile(RUNTIME):
   print("No runtime library found in SDK archive (expected '%s')" % RUNTIME)
   sys.exit(1)

pluginPrefix =  "maya/plug-ins/%s" % maya.Version(nice=True)

excons.DeclareTargets(env, [{"name": "NeuronForMaya",
                             "type": "dynamicmodule",
                             "bldprefix": maya.Version(),
                             "prefix": pluginPrefix,
                             "ext": maya.PluginExt(),
                             "incdirs": ["src", SDKDIR + "/include/NeuronDataReader"],
                             "libdirs": [SDKDIR + "/lib/" + ("win/desktop/x64" if sys.platform == "win32" else "mac")],
                             "srcs": excons.glob("src/*.cpp"),
                             "libs": ["NeuronDataReader"],
                             "install": {pluginPrefix: [RUNTIME],
                                         "maya/scripts": excons.glob("src/*.mel")},
                             "custom": [maya.Require]}])
