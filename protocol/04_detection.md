# Automated detection protocol

#### These are instructions for running batdetect2 using the bat-detector-msds pipeline on recovered nighttime recordings

## Pre-steps:

- Fork `aditya-uw/bat-detector-msds` to access the batdetect2-pipeline branch in that repository.
  - This is where the bash script that runs the pipeline is stored.
- Download the forked repository with `git clone (your forked repository)` on the Linux machine with the mounted UBNA hard drives.
- Follow the forked repository's README.md instructions about `pip install -r requirements.txt` and updating the submodule.
  - It is ideal to install requirements and dependencies in a virtual environment with python=3.x.

## Pipeline steps:
1) Open the cloned repository with `cd ~/bat-detector-msds`
2) Use `git fetch upstream` to update your repository according to the updated pipeline:
   - If there is recovered data you wish to run the pipeline on, check the updated repository's outputs to ensure the pipeline has not already been run on the desired data.
4) Use the field records to find where your recovered nighttime recorder data has been uploaded. For example, `/mnt/ubna_data_02/recover-20230609/UBNA_012`
5) Use the command in this format: `nohup sh scripts/pipeline_for_recovered_deployments.sh "/mnt/ubna_data_02/recover-20230609" "UBNA_012" "true" "true" &`
   - `nohup (command) &` will keep the pipeline running regardless of ssh connection and write the output into a nohup.out file. This file will be stored amidst the files in bat-detector-msds
   - `sh scripts/pipeline_for_recovered_deployments.sh` runs the bash script that runs the detection pipeline. It takes 4 arguments:
      - Upload folder path: `/mnt/ubna_data_02/recover-20230609`
      - Nighttime SD card name: `UBNA_012`
      - String “boolean” whether detector should be run: “true”
      - String “boolean” whether summary figure should be generated: “true”
         - The reason for the booleans is because generating detections is only a 1-time operation but generating figures can be needed multiple times to update figure format using the detections.csv.
6) The detections.csv and figures are saved into `bat-detector-msds/output_dir/recover-DATE/SD_UNIT` .
   - The naming conventions of each file will remember recover-DATE and SD_UNIT to trace back to where the data came from.
7) Once you hit `ENTER` on the above command, you will see this message: `nohup: ignoring input and appending output to 'nohup.out'`
8) Use `CTRL+C` to exit the dialog and use the command `tail -f nohup.out` to view the nohup.out file.
   - The nohup.out file will update dynamically and independently so the command will run even if you `CTRL+C` again to exit viewing.
9) If you wish to cancel the command:
   - Use `ps e` to view the currently running processes; the above command will be linked to python. The PID is the ID of that process displayed to the left of the process.
   - Use `kill PID` where you type the processes' ID in place for PID to cancel the command.
   - **Note: Only cancel the command that you wish to cancel, you can see how much time has passed since each job has been started in `ps e`.** 
  
## Pipeline Outputs
1) Once the pipeline command has been run, some details will be displayed for user sanity:
   - `Input Directory: /mnt/ubna_data_02/recover-20230609/UBNA_012`
   - `Output CSV name: batdetect2_pipeline__recover-20230609_UBNA_012.csv`
   - `Output Directory: output_dir/recover-20230609/UBNA_012`
2) After these details are shown, the pipeline will search the input directory to gather a list of usable audio files to feed into batdetect2.
   - Here, usable is defined as 30-min long audio recordings collected using the Audiomoth's internal microphone.
   - External microphone recordings of 30-min lengths may exist in our input directories but these recordings lack real world data.
   - Other recordings that are 488-bytes may also exist in our input directories and lack real world data.
   - We are still investigating why these "unusable" files are generated by the Audiomoth but our pipeline ignores them using os.stat() and exiftool.
3) The pipeline will then display the number of usable files out of the total number of files in the input directory.
   - This will take some time as the pipeline checks which files are usable in the input directory.
4) The pipeline will then begin generating segments for all usable files.
   - This may take some time but still is a fraction of the time it will take to generate detections.
5) Once segments have been generated, the pipeline will begin displaying a loading bar:
   - Here is an example: `0%|          | 1/7560 [0:00:05<9:29:27,  4.55s/it]`
   - Here, 0% is the rounded percent of segments whose detections have been generated.
   - 1 is the number of segments whose detections have been generated.
   - 7560 is the total number of segments that were generated by the pipeline.
   - 0:00:05 is the time that has passed for generating detections. (HH:MM:SS)
   - 9:29:27 is the time the pipeline still needs to generate detections for all segments. (HH:MM:SS)
   - 4.55s/it is the time it takes to generate detections per segment for our pipeline on our chosen machine.
      - It should take <10s/it for the pipeline. If it takes much longer, consider that multiple pipelines are running at once and need to be cancelled so that 1 pipeline can run at once.
      - As it should take <10s/it, it should also take <17hrs for ~6000 segments. Use this to know if you need to cancel commands. It's better to cancel long processes early than to wait for them to get faster.