# Huawei_FACS
This is basically a step by step tutorial for Huawei FACS.

## Create Huawei Cloud Accout
* Go to http://www.huaweicloud.com/en-us/?locale=en-us
* Click “Register” or “Log In”
* Do registration or logging

## Buy Elastic Cloud Server（FPGA Accelerated）
* Click “Console”, and then “Computing -> Elastic Cloud Server”
* Click “Buy ECS”
* Charge Mode:“On-demand” （Optional）
* Specifications: “FPGA-accelerated | fp1c.2xlarge.15 | 8vCPU” (SDAccel)
* Setup password (for root)
* Buy ECS

## Login ECS
* Click “Console”, and then “Computing -> Elastic Cloud Server”
* Click “More->Start”，after starting completed
* Click “Remote Login” or use ssh to connect to the Elastic IP Address

## Download Huawei development tool suite (fp1)
* git clone https://github.com/Huawei/huaweicloud-fpga.git
    Note: Huawei VU9P dsa is included in the suite – 
		  huaweicloud-fpga/fp1/hardware/sdaccel_design/lib/platform/xilinx_huawei-vu9p-fp1_4ddr-xpr_4_1

## Modifying the Configuration File and Setting up
  ### Compile and install the FPGA image management tool
    * cd huaweicloud-fpga/cli/fisclient
    * python setup.py install
    * cp cfg.file /etc
  ### Modify /etc/cfg.file，Configure 4 parameters:
    * OS_AUTH_URL  -- Open http://developer.hwclouds.com/endpoint.html (Chinese version), Find Endpoint part of “统一身份认证服务（IAM）”,       * configure OS_AUTH_URL as https://iam.cn-north-1.myhwclouds.com
    * OS_FIS_URL – Open http://developer.hwclouds.com/endpoint.html, find Endpoint part of “弹性云服务器（ECS）”，configure it as                  https://ecs.cn-north-1.myhwclouds.com
    * OS_USER_ID – Click right upper user name->Basic Information->Manage my credentials，Find “User ID”
    * OS_TENANT_ID -- Click right upper user name->Basic Information->Manage my credentials，Find “Project ID” corresponding to the                “Region”（such as cn-north-1）
   ### Save /etc/cfg.file，check correctness by fischeck command
        Note：the password needed is the cloud account password
   ### Install s3cmd tool
    * easy_install s3cmd
        Note: Make sure there’s no “setup.cfg” file in the working directory.
   ### Get OBS configurations:
    * Go to http://developer.huaweicloud.com/dev/endpoint， Find “区域” and “Endpoint” informations for “对象存储服务（OBS）”
   ### Get Access Keys -- Click right upper user name->Basic Information->Manage my credentials，click “Access Keys” -> “Add Access Key”
   ### Run “s3cmd --configure”
   ### Check /root/.s3cfg file
   ### Create OBS bucket – click console->Object Storage Service, then “Create Bucket” ，choose a unique name for Bucket name, such as “obs-Xilinx”
   ### Modify huaweicloud-fpga/fp1/setup.cfg:
      FPGA_DEVELOP_MODE=“sdx”	
		  XILINX_LIC_SETUP=“2100@100.125.1.240:2100@100.125.1.251”    	
		  VIVADO_VER_REQ=“2017.1”
   ### source huaweicloud-fpga/fp1/setup.sh
      Note: This step is necessary when starting new terminal.
   ### Install Fpga Management tool:
      cd huaweicloud-fpga/fp1
		  bash fpga_tool_setup.sh
   ### Generate xclbin file, for example:
      cd huaweicloud-fpga/fp1/hardware/sdaccel_design/examples/mmult_hls/scripts
		  sh compile.sh hw
   ### Register FPGA Image:
      edit AEI_Register.cfg file: MODE=OCL, OBS_BUCKETNAME=obs-Xilinx
		  sh AEI_Register.sh –n “ocl-test” –d “ocl-desc”
      After the registration, an ID is assigned to the FPGA image. Please record this ID, because it can be used to query the     registration status, and load, delete, and associate the image. Or
   ### Get FPGA image ID by fisclient 
      fisclient 
      fis fpga-image-list
 
## Load FPGA Image
* Confirm there’s FPGA Load tool FpgaCmdEntry by command “ls /usr/local/bin/FpgaCmdEntry”
* Confirm there’s FPGA in ECS by command “FpgaCmdEntry DF -D”. “Fpga DPDK Device” for “Type” means that there’s high performance architecture (DPDK) device in ECS. “Fpga OCL Device” means that there’s general purpose architecture (OCL) device in ECS.
* Check if FPGA image is loaded in the queried slot by command “FpgaCmdEntry IF –S slot_fpga”. “NOT _PROGRAMMED” for “LoadStatusName” means that FPGA image is not loaded. 
* Load FPGA Image to device by command “FpgaCmdEntry LF –S slot_fpga –I AEI_id”. “slot_fpga” is FPGA slot number, and “AEI_id” is the compiled FPGA Image ID. 
* Check if load image success by command “FpgaCmdEntry IF –S slot_fpga”. If “LoadStatusName” is “LOADED”, it means the device is successfully loaded.

## Compile & Execute SDAccel example
### Compile host program
* source huaweicloud-fpga/fp1/setup.sh 
* cd huaweicloud-fpga/fp1/software/app/sdaccel_app/mmult_hls
* make
### Generate xclbin
* source huaweicloud-fpga/fp1/setup.sh 
* cd huaweicloud-fpga/fp1/hardware/sdaccel_design/examples/mmult_hls/scripts
* sh compile.sh hw
	  Note: This step has been done before registering FPGA image.
### Perform hardware test
* cd huaweicloud-fpga/fp1/software/app/sdaccel_app/mmult_hls
* sh run.sh mmult <bin_dir>/bin_mmult_hw.xclbin

## Reference
* Help Center -- http://support.huaweicloud.com/en-us/index.html
* Elastic Cloud Server User Guide -- http://static.huaweicloud.com/upload/files/pdf/20170901/20170901163324_63827.pdf
