# PowershellWPFgui

Making GUI in Powershell is terrible if you deside to use WinForms. 

You should try making it in WPF!

To do this you just create your desired GUI in Visual Studio or another WPF editor, and puts it in a variable like this 
(could also be put in separate files and just get-content or similar).

-------------

$inputXML1 = @"
<Window x:Name="MainWindow1" x:Class="WpfApp1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfApp1"
        mc:Ignorable="d"
        Title="Title" Height="200" Width="300" ResizeMode="NoResize" Topmost="True" WindowStartupLocation="CenterScreen" BorderBrush="Black">
    <Grid Height="169" VerticalAlignment="Bottom">
        <Button x:Name="Next1" Content="Next" HorizontalAlignment="Left" Height="20" Margin="197,139,0,0" VerticalAlignment="Top" Width="40" BorderBrush="Black" FontSize="11"/>
        <Button x:Name="Cancel1" Content="Cancel" HorizontalAlignment="Left" Height="20" Margin="242,139,0,0" VerticalAlignment="Top" Width="40" BorderBrush="Black" FontSize="11"/>
        <Label Content="Title" HorizontalAlignment="Left" Height="28" Margin="10,10,0,0" VerticalAlignment="Top" Width="193" AutomationProperties.Name="Welcome to Title" FontSize="14"/>
        <TextBlock HorizontalAlignment="Left" Margin="10,43,0,0" TextWrapping="Wrap" Text="textnr2." VerticalAlignment="Top" Height="71" Width="227" Padding="5,0,0,0" ScrollViewer.CanContentScroll="True"/>
    </Grid>
</Window>
"@   

------------------------------------------

If you want several windows you can just create more.
You then run the following code snippet, it will create objects from all named elements and store them in variables

    $XMLs = get-variable -Name "inputXML*"
    Foreach ($XML in $XMLs) 
    {
        $Number = $XML.Name.Substring($XML.Name.Length -1 )
        New-Variable -Name "Form$Number" -Value "Form$Number"
     
        $XML = $XML.value -replace 'mc:Ignorable="d"','' -replace "x:N",'N'  -replace '^<Win.*', '<Window'

        [void][System.Reflection.Assembly]::LoadWithPartialName('presentationframework')
        [xml]$XAML = $XML

        $reader=(New-Object System.Xml.XmlNodeReader $xaml)
        try
        { 
            set-Variable -Name "Form$Number" -value ([Windows.Markup.XamlReader]::Load( $reader ))
        }
            
        catch
        {
            Write-Warning "Unable to parse XML, with error: $($Error[0])`n Ensure that there are NO SelectionChanged properties (PowerShell cannot process them)"
            throw
        }

You can then interact with the objects like this

    $WPFNext1.Add_Click({
        $Form2.ShowDialog() | out-null
        $form1.Close()
    })
    $WPFCancel1.Add_Click({
        param([object]$sender,[System.EventArgs]$e)
        $UserCredentials.status = [System.Windows.Forms.DialogResult]::Cancel
        $form1.Close()   
    })

Finaly you can show the for like this:

$Form1.ShowDialog() 

From what I have seen there is very little limitations to this (as you are using the presentationframework), its so much easier to work with then forms simply because you get most of the structure from the XAML. And in the case you need any forms functionality you can combine it 😉
