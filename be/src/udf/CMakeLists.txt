# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.


# where to put generated libraries
set(LIBRARY_OUTPUT_PATH "${BUILD_OUTPUT_ROOT_DIRECTORY}/udf")

# where to put generated binaries
set(EXECUTABLE_OUTPUT_PATH "${BUILD_OUTPUT_ROOT_DIRECTORY}/udf")

set(UDF_IR_SRC_FILES udf-ir.cc)
add_library(UdfIr ${UDF_IR_SRC_FILES})
add_dependencies(UdfIr gen-deps)

set(UDF_SRC_FILES
  udf.cc
  udf-test-harness.cc
)

# Build this library twice. Once to be linked into the main impalad. This version
# can have dependencies on our other libs. The second version is shipped as part
# of the UDF sdk, which can't use other libs.
# Note that '${UDF_IR_SRC_FILES}' is not needed in 'Udf' because it will be linked
# together with 'UdfIr' as part of the Impala binary, but 'ImpalaUdf' needs it because it
# is shipped separately.
add_library(Udf ${UDF_SRC_FILES})
add_dependencies(Udf gen-deps)

add_library(ImpalaUdf ${UDF_SRC_FILES} ${UDF_IR_SRC_FILES})
add_dependencies(ImpalaUdf gen-deps)
set_target_properties(ImpalaUdf PROPERTIES COMPILE_FLAGS "-DIMPALA_UDF_SDK_BUILD")

if (BUILD_WITH_NO_TESTS)
  return()
endif()

ADD_UDF_TEST(udf-test)
ADD_UDF_TEST(uda-test)
