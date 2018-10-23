# AzurePowershell
##Copy file from Blob storage to Azure data lake store 

Function Global:Copy-BlobToAdls($SourceFilePath,$DestinationFilePath){

    #Downloading the Blob File
    $FileDownloadPath = 'FILE_DOWNLOAD_PATH'
    $Ctx = New-AzureStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey
    Get-AzureStorageBlobContent -Destination $FileDownloadPath -Context $Ctx -Blob $SourceFilePath  -Container $ContainerName -Force 
	   
    #Adding header in the file
    $LocalDownloadedFilePath = $FileDownloadPath+'\'+$SourceFilePath
    if(Test-Path $LocalDownloadedFilePath)
{

        #ADLS FILE CHECK and UPLOAD
        $DestFileCheck = Test-AzureRmDataLakeStoreItem -Account $AdlsName  -Path $DestinationFilePath
        IF($DestFileCheck -eq $True)
        {
            Remove-AzureRmDataLakeStoreItem -AccountName $AdlsName -Path $DestinationFilePath -Force
            Import-AzureRmDataLakeStoreItem -AccountName $AdlsName -Path $LocalDownloadedFilePath -Destination $DestinationFilePath

        }else
        {
            Import-AzureRmDataLakeStoreItem -AccountName $AdlsName -Path $LocalDownloadedFilePath -Destination $DestinationFilePath
        
        }

        $MigratedFiles =  @{} 
        $FilesDetails = Get-AdlStoreItem -Account $AdlsName  -Path $DestinationFilePath
        Add-Content -Value "$TimeinSec Added the List of Migrated Files (With Headers) to  $AllMigratedFilesPath" -Path $LogFile
        Add-Content -Value $ListofMigratedFiles -Path $LogFile
        remove-item $LocalDownloadedFilePath
        	
    }
}
