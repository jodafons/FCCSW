################################################################################
# Package: RecCalorimeter
################################################################################
gaudi_subdir(RecCalorimeter v1r0)

# this declaration will not be needed in the future
gaudi_depends_on_subdirs(GaudiAlg FWCore Detector/DetInterface Detector/DetSegmentation Detector/DetCommon Reconstruction/RecInterface)

find_package(FastJet)
find_package(ROOT COMPONENTS Physics Tree)
find_package(FCCEDM)
find_package(PODIO)
find_package(HepMC)
find_package(DD4hep)
find_package(Geant4)

set(CMAKE_MODULE_PATH  ${CMAKE_MODULE_PATH}  ${DD4hep_ROOT}/cmake )
include( DD4hep )

gaudi_add_module(RecCalorimeterPlugins
                 src/components/*.cpp
                 INCLUDE_DIRS FastJet ROOT FWCore HepMC FCCEDM PODIO DD4hep DetInterface DetSegmentation Geant4 DetCommon RecInterface RecCalorimeter
                 LINK_LIBRARIES FWCore Fastjet ROOT GaudiAlgLib FCCEDM PODIO HepMC DD4hep DetSegmentation Geant4 DetCommon)


install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/tests/options DESTINATION ${CMAKE_INSTALL_DATADIR}/${CMAKE_PROJECT_NAME}/Reconstruction/RecCalorimeter)

include(CTest)
gaudi_add_test(genJetClustering
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               FRAMEWORK tests/options/genJetClustering.py)

gaudi_add_test(simulateECalSimple
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               FRAMEWORK tests/options/geant_fullsim_ecalSimple_singleparticles.py)

gaudi_add_test(EcalReconstructionNoiseFromFile
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               DEPENDS simulateECalSimple
               FRAMEWORK tests/options/runEcalSimple_ReconstructionSW_noiseFromFile.py
               PASSREGEX "Segmentation cells  \\(Nphi, Neta, minEta\\): \\[629, 331, 1\\]" )

# TODO: causes issues with CI, investigate
#gaudi_add_test(EcalReconstructionCheckNumClusters
#               ENVIRONMENT PYTHONPATH+=${PODIO_PYTHON_DIR}
#               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
#               COMMAND python Reconstruction/RecCalorimeter/tests/scripts/ecalRecoCheckNumClusters.py
#               DEPENDS EcalReconstructionNoiseFromFile)

gaudi_add_test(simulateHCal
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               FRAMEWORK tests/options/geant_fullsim_hcal_singleparticles.py)

gaudi_add_test(HcalDigitisationWithoutNoise
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               DEPENDS simulateHCal
               FRAMEWORK tests/options/runHcalDigitisationWithoutNoise.py)

gaudi_add_test(HcalDigitisationFlatNoise
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               DEPENDS simulateHCal
               FRAMEWORK tests/options/runHcalDigitisationFlatNoise.py)

gaudi_add_test(simulateFullCaloSystem
               WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
               FRAMEWORK tests/options/runFullCaloSystem_SimAndDigitisation.py)

gaudi_add_test(reconstructionFullCaloSystem
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                DEPENDS simulateFullCaloSystem
                FRAMEWORK tests/options/runFullCaloSystem_ReconstructionSW_noNoise.py)

gaudi_add_test(reconstructionFullCaloSystemWithNoise
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                DEPENDS simulateFullCaloSystem
                FRAMEWORK tests/options/runFullCaloSystem_ReconstructionSW_noiseFromFile.py
                PASSREGEX "Segmentation cells  \\(Nphi, Neta, minEta\\): \\[704, 335, 1\\]")

gaudi_add_test(correctionFullCaloSystem
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                DEPENDS reconstructionFullCaloSystem
                FRAMEWORK tests/options/runFullCaloSystem_CorrectECalBarrelSliWinClusters.py)

gaudi_add_test(estimatePileup
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                FRAMEWORK tests/options/runPreparePileup.py)

gaudi_add_test(recTopoClustersBarrelCaloSystem
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                DEPENDS simulateFullCaloSystem
                FRAMEWORK tests/options/runBarrelCaloSystem_ReconstructionTopoClusters_noNoise.py)

gaudi_add_test(recTopoClustersBarrelCaloSystemElecNoise
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                DEPENDS simulateFullCaloSystem
                FRAMEWORK tests/options/runBarrelCaloSystem_ReconstructionTopoClusters_electrNoise.py)

gaudi_add_test(recSplitTopoClustersBarrelCaloSystemElecNoiseConeSelected
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                DEPENDS simulateFullCaloSystem
		FRAMEWORK tests/options/runBarrelCaloSystem_ReconstructionTopoClusters_electrNoise_coneSelection.py)
