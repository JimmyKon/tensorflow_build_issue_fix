# tensorflow_build_issue_fix
# How to fix dyld: Library not loaded: @rpath/libcudart.7.5.dylib issue when you build tensorflow on your Mac


My compute:
2012 early iMac
CPU:i5
Display Card: GT640M

Xcode 7.3
CUDA 7.5.27
CUDNN 4


Error Message:

>ERROR: /Users/\*\*\*\*\*/tensorflow/tensorflow/cc/BUILD:179:1: Executing genrule //tensorflow/cc:data_flow_ops_genrule failed: bash failed: error executing command /bin/bash -c ... (remaining 1 argument(s) skipped): com.google.devtools.build.lib.shell.AbnormalTerminationException: Process terminated by signal 5.
dyld: Library not loaded: @rpath/libcudart.7.5.dylib
  Referenced from: /private/var/tmp/_bazel\_\*\*\*\*\*/c8b39b0fb8f012cc5094900afcbdd9c4/execroot/tensorflow/bazel-out/host/bin/tensorflow/cc/ops/data_flow_ops_gen_cc
  Reason: image not found
/bin/bash: line 1: 15306 Trace/BPT trap: 5       bazel-out/host/bin/tensorflow/cc/ops/data_flow_ops_gen_cc bazel-out/local_darwin-opt/genfiles/tensorflow/cc/ops/data_flow_ops.h bazel-out/local_darwin-opt/genfiles/tensorflow/cc/ops/data_flow_ops.cc 0
Target //tensorflow/cc:tutorials_example_trainer failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 2855.945s, Critical Path: 2810.59s

let's add --verbose_failures and build again.

I found the genrule-setup.sh file will execute before error.

	...execroot/tensorflow/external/bazel_tools/tools/genrule/genrule-setup.sh

ok, print file timestamp first.

	stat genrule-setup.sh
	
output like this:

	16777217 56288053 -rwxr-xr-x 1 ****** wheel 0 242 "Sep  4 23:26:23 2016" "Sep  2 22:34:23 2026" "Sep  4 22:34:24 2016" "Sep  4 22:34:21 2016" 4096 8 0 genrule-setup.sh
	
"Sep  2 22:34:23 2026"? yes, record this timestamp.

open this file, add the environment configuration to the end of the file

	export DYLD_LIBRARY_PATH=/usr/local/cuda/lib
	
and then, recover genrule-setup.sh timestamp

	touch -t YYYYMMDDhhmm.SS genrule-setup.sh

YYYYMMDDhhmm.SS is your recorded timestamp,my situation is 202609022234.23


compile again, done.

	bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

