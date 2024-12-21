##############################################################
#muiltiple RVTools CSV Summary
##############################################################
# 
# This script will search for and grab vInfo CSVs and check for corresponding
# vHosts tabs to create a per CLUSTER summary sizing report for NUTANIX
# 
#
#######Clear stuff up and set stuff up
    $report_variable = New-Object System.Collections.ArrayList
    $export_filename = "export-report.csv"
    $vCenter_vinfo_file = ""
    $vcenter_hwInfo_file = ""
    $vCenter_vInfo_Data = ""
    $basepath = get-location 

#Stuff
Foreach($vCenter_vinfo_file in gci -path $basepath -Recurse | where {$_.Name -eq 'RVTools_tabvInfo.csv'})
{
    #### VINFO #####
    #Grab each file, find location, search for vHost matching file based on VDI SDK Server data field
    $vcenter_vinfo_data = ipcsv $vcenter_vinfo_file.PSPath
    
    ##### Check for VHOSTS so we can collect / compare HW ####
    if($vcenter_vhost_file = ls $vCenter_vinfo_file.Directory | where {$_.Name -eq 'RVTools_tabvHost.csv'})
        {
            #import working file into variable
            $vcenter_vhost_data = ipcsv $vcenter_vhost_file.PSPath
        }
    Else
        {
           #section to handle if no matching file is found..
           wh "no corresponding vhost file found"
           $row.HWCores = 0
           $row.Hosts = 0
        }
    

    ### BY CLUSTER ####
    $clusters = $vCenter_vinfo_data | select Cluster -Unique
    Foreach($cluster in $clusters)
    {
            $cluster_vms = $vCenter_vInfo_data | where {$_.Cluster -eq $cluster.cluster}
            if($vcenter_vhost_data)
                {
                    $cluster_hw = $vcenter_vhost_data | where {$_.cluster -eq $cluster.cluster}
                    $cluster_cores = ($cluster_hw | Measure-Object -Property "# Cores" -Sum).sum
                }
                #$ratio = [Math]::Round($row.vCPUs / $row.HwCores, 0)
                
                #Establish CSV volume headers and insert values.
                $row = "" | Select vCenter, Cluster, vCPUs, vRAMGiB, vProvGiB, vInUseGiB, HWCores, Hosts, CoresPerHost, vCPU:Core
                $row.vCenter = ($vCenter_vInfo_data[1].'VI SDK Server').split(".")[0]
                $row.Cluster = $cluster.cluster
                $row.vCPUs = ($cluster_vms | Measure-Object -Property "CPUs" -Sum).Sum
                $row.vRAMGiB = [Math]::Round((($cluster_vms | Measure-Object -Property "Memory" -Sum).Sum/1024), 0)
                $row.vProvGiB = [Math]::Round((($cluster_vms | Measure-Object -Property "Provisioned MiB" -Sum).Sum/1024), 0)
                $row.vInUseGiB = [Math]::Round((($cluster_vms | Measure-Object -Property "In Use MiB" -Sum).Sum/1024), 0)
                $row.HWCores = $cluster_cores
                $row.Hosts = $cluster_hw.count
                $row.CoresPerHost = [Math]::Round(($row.HWCores/$row.Hosts), 1)
                $row."vCPU:Core" =  ([Math]::Round($row.vCPUs / $row.HwCores, 1)).Tostring() + ":1"
           $report_variable.Add($row)
    }

}
$report_variable | epcsv $export_filename -NoTypeInformation
