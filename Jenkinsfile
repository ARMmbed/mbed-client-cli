// List of targets to compile
def morpheusTargets = [
  //"LPC1768",
  //"NUCLEO_F401RE",
  "NRF51DK",
  "K64F"
  ]
  
// Map morpheus toolchains to compiler labels on Jenkins
def toolchains = [
  ARM: "armcc",   // open issue: https://github.com/ARMmbed/mbed-os/issues/125
  // IAR: "iar_arm", // open issue: https://github.com/ARMmbed/mbed-os/issues/357
  GCC_ARM: "arm-none-eabi-gcc"
  ]
  
// Initial maps for parallel build steps
def stepsForParallel = []

// Jenkins pipeline does not support map.each, we need to use oldschool for loop
for (int i = 0; i < morpheusTargets.size(); i++) {
  for(int j = 0; j < toolchains.size(); j++) {
    def target = morpheusTargets.get(i)
    def toolchain = toolchains.keySet().asList().get(j)
    def compilerLabel = toolchains.get(toolchain)
    def stepName = "mbed-os5:${target} ${toolchain}"
    stepsForParallel[stepName] = morpheusBuildStep(target, compilerLabel, toolchain)
    stepName = "mbed-os3:${target} ${toolchain}"
    stepsForParallel[stepName] = yottaBuildStep(target, compilerLabel, toolchain)
  }
}
stepsForParallel["x86-linux-native"] = yottaTestStep("x86-linux-native")

/* Jenkins does not allow stages inside parallel execution, 
 * https://issues.jenkins-ci.org/browse/JENKINS-26107 will solve this by adding labeled blocks
 */
// Actually run the steps in parallel - parallel takes a map as an argument, hence the above.
stage "build testapps"
parallel stepsForParallel

def execute(cmd) {
    if(isUnix()) {
       sh "${cmd}"
    } else {
       bat "${cmd}"
    }
}

//Create morpheus build steps for parallel execution
def morpheusBuildStep(target, compilerLabel, toolchain) {
  return {
    node ("${compilerLabel}") {
      deleteDir()
      dir("mbed-client-cli") {
        checkout scm
        execute("mbed | grep \"^version\"")
        execute("mbed compile -m ${target} -t ${toolchain} --library")
      }
    }
  }
}
def yottaBuildStep(target, compilerLabel, toolchain) {
  return {
    node ("${compilerLabel}") {
      deleteDir()
      dir("mbed-client-cli") {
        checkout scm
        execute("yotta --version")
        execute("yotta target $target")
        execute("yotta --plain build mbed-client-cli")
      }
    }
  }
}

def yottaTestStep(target) {
  return {
    node ("${compilerLabel}") {
      deleteDir()
      dir("mbed-client-cli") {
        checkout scm
        execute("yotta --version")
        execute("yotta target $target")
        execute("yotta test mbed_client_cli_test")
        //execute("gcov ./build/x86-linux-native/test/CMakeFiles/mbed_client_trace_test.dir/Test.cpp.o")
        execute("lcov --base-directory . --directory . --capture --output-file coverage.info")
        execute("genhtml -o ./test_coverage coverage.info")
        execute("gcovr -x -o junit.xml")
        execute("cppcheck --enable=all --std=c99 --inline-suppr --xml --xml-version=2 -I mbed-client-trace/ source 2> cppcheck.xml")
      }
    }
  }
}

