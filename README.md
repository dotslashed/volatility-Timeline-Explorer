# Volatility-Timeline-Explorer
A Powershell wrapper function that opens volatility3 output csv directly into Eric Zimmerman's TimeLine Explorer  

-> Find Powershell profile path, enter: $PROFILE in Powershell terminal  
-> Open the profile.ps1 file in text editor  
-> Paste the below wrapper in the profile (You can give any name to the wrapper function):  
-> Change your custom paths in the wrapper
```
function Time-Exp {
    <#
    Time-Exp
    ----------
     Takes piped output as input
     Saves as a temp CSV file in your chosen folder
     Opens it with Timeline Explorer (or any EXE you specify)
     Deletes the CSV when the Timeline Explorer is closed (unless -KeepFile is used)
    #>

    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline=$true)]
        $InputObject,

        [switch]$KeepFile
    )

    begin {
        # CUSTOM PATHS
        $ExePath   = "D:\Zimmerman-tools\TimelineExplorer\TimelineExplorer.exe"   # path to Timeline Explorer
        $OutputDir = "D:\test\csvs"                     # where temp CSVs are created and deleted from (Any prefered folder path)

        # Ensure directory exists
        if (-not (Test-Path $OutputDir)) {
            New-Item -ItemType Directory -Path $OutputDir | Out-Null
        }

        # Create unique temp filename (avoids conflicts)
        $script:TempFile = Join-Path $OutputDir ("input_{0}.csv" -f ([guid]::NewGuid()))
    }

    process {
        # Append pipeline objects into CSV
        $InputObject | Export-Csv -Path $script:TempFile -NoTypeInformation -Append
    }

    end {
        try {
            # Start EXE and wait for it to close
            Start-Process -FilePath $ExePath -ArgumentList $script:TempFile -Wait
        }
        finally {
            if (-not $KeepFile) {
                Remove-Item $script:TempFile -Force -ErrorAction SilentlyContinue
            }
            else {
                Write-Host "CSV kept at: $script:TempFile"
            }
        }
    }
}
```

-> Save and restart powershell  
->Volatility3 sample command:
```python .\vol.py -f "D:\memdump\memdump.mem" -r csv windows.pslist | ConvertFrom-Csv | Time-Exp```  
The above command will create a unique guid csv file inside the folder and will be deleted when Timeline Explorer is closed to avoid being populated by CSVs  
The ```-KeepFile``` switch will save the CSVs permanently (Optional)  

Output Image:
<img width="1869" height="974" alt="image" src="https://github.com/user-attachments/assets/474683b2-2f45-4a26-8492-bfc00e8642fe" />  

**Credits:** [Eric Zimmerman Tools](https://ericzimmerman.github.io/#!index.md)

