cmake_minimum_required(VERSION 3.0)

set(TEST_NAME flip)

set(SRCS 
    ${TEST_NAME}.cxx
    )
    
set(BLIF_NAME ${TEST_NAME}.blif)
set(BLOP_NAME ${TEST_NAME}-opt.blif)



add_compile_options(-Wall -std=c++11 -Dblif_name="${BLIF_NAME}")

set(GEN_NAME ${TEST_NAME}-gen)

add_executable(${GEN_NAME} ${SRCS}) 

target_link_libraries(${GEN_NAME} generator)

add_custom_command(OUTPUT ${BLIF_NAME}
  COMMAND ./${GEN_NAME} < /dev/null
  DEPENDS ${GEN_NAME}) 

add_custom_command(OUTPUT ${BLOP_NAME}
  COMMAND python3 ../../optim/abc_optimize.py -i ${BLIF_NAME} -o ${BLOP_NAME} -c ';' -v  # no optimization
  DEPENDS abc ${BLIF_NAME})


set(XML_PARAMS fhe_params.xml)
set(SCRIPT_DIR ../../../runtime/fhe_fv/script)

add_custom_command(OUTPUT ${XML_PARAMS}
        COMMAND bash ${SCRIPT_DIR}/selectParams.sh ${TEST_NAME} ${BLOP_NAME} ${MODEL} ${MIN_SECU}
        DEPENDS ${BLOP_NAME})   

set(LAST_DEPEND ${XML_PARAMS})

add_custom_target(${TEST_NAME} ALL
  DEPENDS ${LAST_DEPEND})

configure_file("run.sh" "run.sh" COPYONLY)
configure_file("ref.txt" "ref.txt" COPYONLY)
