<#
.SYNOPSIS
    PowerShell script with GUI (Windows Forms) for NAS diagnostics and system monitoring via SSH.
#>

Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing
Add-Type -AssemblyName Microsoft.VisualBasic

# Check and auto-install Posh-SSH module if needed
if (-not (Get-Module -ListAvailable -Name Posh-SSH)) {
    $install = [System.Windows.Forms.MessageBox]::Show("The 'Posh-SSH' module is required. Do you want to install it now?", "Installation Required", [System.Windows.Forms.MessageBoxButtons]::YesNo, [System.Windows.Forms.MessageBoxIcon]::Question)
    if ($install -eq [System.Windows.Forms.DialogResult]::Yes) {
        Install-Module -Name Posh-SSH -Force -Scope CurrentUser
    } else {
        exit
    }
}
Import-Module Posh-SSH

# Main window creation
$form = New-Object System.Windows.Forms.Form
$form.Text = "NAS SSH Inspector (v0.6 ENG)"
$form.Size = New-Object System.Drawing.Size(850, 700)
$form.StartPosition = "CenterScreen"
$form.FormBorderStyle = [System.Windows.Forms.FormBorderStyle]::FixedDialog
$form.MaximizeBox = $false

# --- SSH Connection Section ---
$groupBoxConn = New-Object System.Windows.Forms.GroupBox
$groupBoxConn.Text = " SSH Connection Settings "
$groupBoxConn.Location = New-Object System.Drawing.Point(15, 15)
$groupBoxConn.Size = New-Object System.Drawing.Size(805, 80)
$form.Controls.Add($groupBoxConn)

# IP
$labelIp = New-Object System.Windows.Forms.Label
$labelIp.Text = "IP / Host :"
$labelIp.Location = New-Object System.Drawing.Point(15, 30)
$labelIp.AutoSize = $true
$groupBoxConn.Controls.Add($labelIp)

$textIp = New-Object System.Windows.Forms.TextBox
$textIp.Location = New-Object System.Drawing.Point(80, 27)
$textIp.Size = New-Object System.Drawing.Size(120, 20)
$textIp.Text = "192.168.1.100"
$groupBoxConn.Controls.Add($textIp)

# Port
$labelPort = New-Object System.Windows.Forms.Label
$labelPort.Text = "Port :"
$labelPort.Location = New-Object System.Drawing.Point(215, 30)
$labelPort.AutoSize = $true
$groupBoxConn.Controls.Add($labelPort)

$textPort = New-Object System.Windows.Forms.TextBox
$textPort.Location = New-Object System.Drawing.Point(250, 27)
$textPort.Size = New-Object System.Drawing.Size(50, 20)
$textPort.Text = "22"
$groupBoxConn.Controls.Add($textPort)

# User
$labelUser = New-Object System.Windows.Forms.Label
$labelUser.Text = "Username :"
$labelUser.Location = New-Object System.Drawing.Point(315, 30)
$labelUser.AutoSize = $true
$groupBoxConn.Controls.Add($labelUser)

$textUser = New-Object System.Windows.Forms.TextBox
$textUser.Location = New-Object System.Drawing.Point(385, 27)
$textUser.Size = New-Object System.Drawing.Size(90, 20)
$textUser.Text = "root"
$groupBoxConn.Controls.Add($textUser)

# Password
$labelPass = New-Object System.Windows.Forms.Label
$labelPass.Text = "Password :"
$labelPass.Location = New-Object System.Drawing.Point(490, 30)
$labelPass.AutoSize = $true
$groupBoxConn.Controls.Add($labelPass)

$textPass = New-Object System.Windows.Forms.TextBox
$textPass.Location = New-Object System.Drawing.Point(575, 27)
$textPass.Size = New-Object System.Drawing.Size(100, 20)
$textPass.UseSystemPasswordChar = $true
$groupBoxConn.Controls.Add($textPass)

# Connect Button
$btnConnect = New-Object System.Windows.Forms.Button
$btnConnect.Text = "Connect"
$btnConnect.Location = New-Object System.Drawing.Point(685, 25)
$btnConnect.Size = New-Object System.Drawing.Size(105, 25)
$btnConnect.BackColor = [System.Drawing.Color]::FromArgb(76, 175, 80)
$btnConnect.ForeColor = [System.Drawing.Color]::White
$groupBoxConn.Controls.Add($btnConnect)

# --- NAS Commands Section ---
$groupBoxCmd = New-Object System.Windows.Forms.GroupBox
$groupBoxCmd.Text = " Diagnostics & Monitoring "
$groupBoxCmd.Location = New-Object System.Drawing.Point(15, 105)
$groupBoxCmd.Size = New-Object System.Drawing.Size(805, 115)
$form.Controls.Add($groupBoxCmd)

# Quick Commands ComboBox
$labelQuick = New-Object System.Windows.Forms.Label
$labelQuick.Text = "Cmd :"
$labelQuick.Location = New-Object System.Drawing.Point(15, 25)
$labelQuick.AutoSize = $true
$groupBoxCmd.Controls.Add($labelQuick)

$comboCmds = New-Object System.Windows.Forms.ComboBox
$comboCmds.Location = New-Object System.Drawing.Point(55, 22)
$comboCmds.Size = New-Object System.Drawing.Size(325, 25)
$comboCmds.DropDownStyle = [System.Windows.Forms.ComboBoxStyle]::DropDownList

$commands = [ordered]@{
    'Disk Space (df -h)' = 'df -h'
    'Detailed Disk List (lsblk)' = 'lsblk -d -o NAME,MODEL,SERIAL,SIZE'
    'Check Disk Acoustic Management (hdparm)' = 'CUSTOM_HDPARM_CHECK'
    'SMART Status of Selected Disk (smartctl)' = 'CUSTOM_SMART_CHECK'
    'Temperature of Selected Disk (smartctl -A)' = 'CUSTOM_TEMP_CHECK'
    'Memory & RAM (free -h)' = 'free -h'
    'CPU Load & Uptime (uptime)' = 'uptime && top -b -n 1 | head -n 5'
    'Active Services (systemctl)' = 'systemctl list-units --type=service --state=running'
}

foreach ($key in $commands.Keys) {
    [void]$comboCmds.Items.Add($key)
}
$comboCmds.SelectedIndex = 0
$groupBoxCmd.Controls.Add($comboCmds)

$btnRunQuick = New-Object System.Windows.Forms.Button
$btnRunQuick.Text = "Run"
$btnRunQuick.Location = New-Object System.Drawing.Point(390, 20)
$btnRunQuick.Size = New-Object System.Drawing.Size(80, 27)
$btnRunQuick.Enabled = $false
$groupBoxCmd.Controls.Add($btnRunQuick)

# Target Disk Selection
$labelDisk = New-Object System.Windows.Forms.Label
$labelDisk.Text = "Disk :"
$labelDisk.Location = New-Object System.Drawing.Point(480, 25)
$labelDisk.AutoSize = $true
$groupBoxCmd.Controls.Add($labelDisk)

$comboDisks = New-Object System.Windows.Forms.ComboBox
$comboDisks.Location = New-Object System.Drawing.Point(530, 22)
$comboDisks.Size = New-Object System.Drawing.Size(85, 25)
$comboDisks.DropDownStyle = [System.Windows.Forms.ComboBoxStyle]::DropDownList
$groupBoxCmd.Controls.Add($comboDisks)

$btnRefreshDisk = New-Object System.Windows.Forms.Button
$btnRefreshDisk.Text = "Refresh"
$btnRefreshDisk.Location = New-Object System.Drawing.Point(625, 20)
$btnRefreshDisk.Size = New-Object System.Drawing.Size(75, 27)
$btnRefreshDisk.Enabled = $false
$groupBoxCmd.Controls.Add($btnRefreshDisk)

# Custom Command
$labelCustom = New-Object System.Windows.Forms.Label
$labelCustom.Text = "Custom :"
$labelCustom.Location = New-Object System.Drawing.Point(15, 68)
$labelCustom.AutoSize = $true
$groupBoxCmd.Controls.Add($labelCustom)

$textCustom = New-Object System.Windows.Forms.TextBox
$textCustom.Location = New-Object System.Drawing.Point(100, 65)
$textCustom.Size = New-Object System.Drawing.Size(280, 20)
$groupBoxCmd.Controls.Add($textCustom)

$btnRunCustom = New-Object System.Windows.Forms.Button
$btnRunCustom.Text = "Send"
$btnRunCustom.Location = New-Object System.Drawing.Point(390, 63)
$btnRunCustom.Size = New-Object System.Drawing.Size(80, 27)
$btnRunCustom.Enabled = $false
$groupBoxCmd.Controls.Add($btnRunCustom)

# --- Console / Output Section ---
$groupBoxOutput = New-Object System.Windows.Forms.GroupBox
$groupBoxOutput.Text = " Console / SSH Output "
$groupBoxOutput.Location = New-Object System.Drawing.Point(15, 235)
$groupBoxOutput.Size = New-Object System.Drawing.Size(805, 410)
$form.Controls.Add($groupBoxOutput)

$textOutput = New-Object System.Windows.Forms.TextBox
$textOutput.Location = New-Object System.Drawing.Point(15, 25)
$textOutput.Size = New-Object System.Drawing.Size(775, 370)
$textOutput.Multiline = $true
$textOutput.ScrollBars = "Vertical"
$textOutput.ReadOnly = $true
$textOutput.BackColor = [System.Drawing.Color]::FromArgb(30, 30, 30)
$textOutput.ForeColor = [System.Drawing.Color]::FromArgb(0, 255, 0)
$textOutput.Font = New-Object System.Drawing.Font("Consolas", 9.5)
$groupBoxOutput.Controls.Add($textOutput)

# Global variable to store SSH session ID
$global:sshSessionId = $null

function Write-Console($message) {
    $textOutput.AppendText("$message`r`n")
    $textOutput.SelectionStart = $textOutput.Text.Length
    $textOutput.ScrollToCaret()
}

# Function to load disk list into dropdown menu
function Update-DiskDropdown {
    if ($global:sshSessionId -ne $null) {
        $password = $textPass.Text
        $cmd = "echo '$password' | sudo -S -p '' lsblk -d -o NAME -n"
        $result = Invoke-SSHCommand -SessionId $global:sshSessionId -Command $cmd
        
        $comboDisks.Items.Clear()
        if ($result.Output) {
            foreach ($line in $result.Output) {
                $diskName = $line.Trim()
                if (-not [string]::IsNullOrWhiteSpace($diskName) -and ($diskName -match "^sd|^nvme|^hd")) {
                    [void]$comboDisks.Items.Add($diskName)
                }
            }
            if ($comboDisks.Items.Count -gt 0) {
                $comboDisks.SelectedIndex = 0
                Write-Console "[Info] Disks detected: $($comboDisks.Items -join ', ')"
            }
        }
    }
}

# Connect / Disconnect button management
$btnConnect.Add_Click({
    if ($global:sshSessionId -eq $null) {
        $ip = $textIp.Text
        $port = [int]$textPort.Text
        $user = $textUser.Text
        $pass = $textPass.Text

        try {
            $secPass = ConvertTo-SecureString $pass -AsPlainText -Force
            $credential = New-Object System.Management.Automation.PSCredential ($user, $secPass)
            
            Write-Console "[Info] Connecting to ${ip}:${port}..."
            $session = New-SSHSession -ComputerName $ip -Port $port -Credential $credential -AcceptKey -ErrorAction Stop
            
            if ($session) {
                $global:sshSessionId = $session.SessionId
                $btnConnect.Text = "Disconnect"
                $btnConnect.BackColor = [System.Drawing.Color]::FromArgb(244, 67, 54)
                $btnRunQuick.Enabled = $true
                $btnRunCustom.Enabled = $true
                $btnRefreshDisk.Enabled = $true
                Write-Console "[Success] Connected to NAS successfully!"
                
                Update-DiskDropdown
            }
        } catch {
            [System.Windows.Forms.MessageBox]::Show("Connection Error: $_", "SSH Error", [System.Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Error)
            Write-Console "[Error] Connection failed: $_"
        }
    } else {
        try {
            Remove-SSHSession -SessionId $global:sshSessionId -ErrorAction SilentlyContinue
            $global:sshSessionId = $null
            $btnConnect.Text = "Connect"
            $btnConnect.BackColor = [System.Drawing.Color]::FromArgb(76, 175, 80)
            $btnRunQuick.Enabled = $false
            $btnRunCustom.Enabled = $false
            $btnRefreshDisk.Enabled = $false
            $comboDisks.Items.Clear()
            Write-Console "[Info] Disconnected from NAS."
        } catch {
            $global:sshSessionId = $null
            $btnConnect.Text = "Connect"
            $btnConnect.BackColor = [System.Drawing.Color]::FromArgb(76, 175, 80)
            $btnRunQuick.Enabled = $false
            $btnRunCustom.Enabled = $false
            $btnRefreshDisk.Enabled = $false
        }
    }
})

function Run-SSHCommand($cmd) {
    if ($global:sshSessionId -ne $null) {
        if ($cmd -match "sudo") {
            $password = $textPass.Text
            $cleanCmd = $cmd -replace "sudo\s+", ""
            $cmd = "echo '$password' | sudo -S -p '' $cleanCmd"
        }

        Write-Console "`n`$ $cmd"
        $result = Invoke-SSHCommand -SessionId $global:sshSessionId -Command $cmd
        if ($result.Output) {
            foreach ($line in $result.Output) {
                Write-Console $line
            }
        }
        if ($result.Error) {
            $errFiltered = $result.Error -replace "\[sudo\s*password\s*for.*?\]", ""
            if (-not [string]::IsNullOrWhiteSpace($errFiltered)) {
                Write-Console "[SSH Error] $errFiltered"
            }
        }
    }
}

$btnRefreshDisk.Add_Click({
    Update-DiskDropdown
})

$btnRunQuick.Add_Click({
    $selectedKey = $comboCmds.SelectedItem
    $mappedValue = $commands[$selectedKey]

    if ($global:sshSessionId -eq $null) { return }

    $selectedDisk = $comboDisks.SelectedItem

    if ($mappedValue -eq "CUSTOM_HDPARM_CHECK") {
        if (-not $selectedDisk) { [System.Windows.Forms.MessageBox]::Show("Please select a disk from the dropdown menu.", "Warning", [System.Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Warning); return }
        $cmd = "sudo hdparm -I /dev/$selectedDisk | grep -i acoustic"
        Run-SSHCommand $cmd
    }
    elseif ($mappedValue -eq "CUSTOM_SMART_CHECK") {
        if (-not $selectedDisk) { [System.Windows.Forms.MessageBox]::Show("Please select a disk from the dropdown menu.", "Warning", [System.Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Warning); return }
        $cmd = "sudo smartctl -H /dev/$selectedDisk"
        Run-SSHCommand $cmd
    }
    elseif ($mappedValue -eq "CUSTOM_TEMP_CHECK") {
        if (-not $selectedDisk) { [System.Windows.Forms.MessageBox]::Show("Please select a disk from the dropdown menu.", "Warning", [System.Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Warning); return }
        $cmd = "sudo smartctl -A /dev/$selectedDisk | grep -i temperature"
        Run-SSHCommand $cmd
    }
    else {
        if ($mappedValue) {
            Run-SSHCommand $mappedValue
        }
    }
})

$btnRunCustom.Add_Click({
    $cmdToRun = $textCustom.Text
    if (-not [string]::IsNullOrWhiteSpace($cmdToRun)) {
        Run-SSHCommand $cmdToRun
    }
})

[void]$form.ShowDialog()
