# Fiji Macro: ND2 to TIFF Batch Converter  

## Overview  
This Fiji macro batch-processes Nikon `.nd2` files in a chosen directory.  
It uses **Bio-Formats** to import each image series and saves them as `.tif` files in organized subfolders.  

## Steps  
- Scans a user-selected directory for `.nd2` files.  
- Imports each file with Bio-Formats, handling multiple series within a single file.  
- Creates subdirectories named after each `.nd2` file.  
- Saves each series as a `.tif` image stack.  

## Usage  
1. Open Fiji.  
2. Run the macro (`Plugins > Macros > Run...`).  
3. Select the folder containing your `.nd2` files when prompted.  
4. The macro will process all `.nd2` files, saving the output `.tif` files into corresponding subfolders.  

## Output  
- Each `.nd2` file generates a folder with the same name.  
- Each series within that file is saved as:  

## Macro Script  

```java
dir = getDirectory("Choose a Directory");
files = getFileList(dir);

setBatchMode(true);
k=0;
n=0;

run("Bio-Formats Macro Extensions");
for(f=0; f<files.length; f++) {
  if(endsWith(files[f], "nd2" )) {
  	k++;
  	id = dir+files[f];
  	Ext.setId(id);
  	Ext.getSeriesCount(seriesCount);
  	print(seriesCount+" series in "+id);
  	n+=seriesCount;
  	for (i=0; i<seriesCount; i++) {
  		run("Bio-Formats Importer", "open=["+id+"] color_mode=Default view=Hyperstack stack_order=XYCZT series_"+(i+1));
  		fullName	= getTitle();
  		dirName 	= substring(fullName, 0,lastIndexOf(fullName, "nd2"));
  		fileName 	= substring(fullName, lastIndexOf(fullName, " - ")+3, lengthOf(fullName));
  		File.makeDirectory(dir+File.separator+dirName+File.separator);

  		print("Saving "+fileName+" under "+dir+File.separator+dirName);
  		
  		getDimensions(x,y,c,z,t);
  		
  		if(false) {
  			save_string = getSaveString(pad);
  			print(dir+File.separator+dirName+File.separator+fileName+"_"+save_string+".tif");
  			run("Image Sequence... ", "format=TIFF name=["+fileName+"] digits="+pad+" save=["+dir+File.separator+dirName+File.separator+"]");
  			
  		} else {
  			saveAs("tiff", dir+File.separator+dirName+File.separator+fileName+"_"+(i+1)+".tif");
  		}
  		run("Close All");
  	}
  }
}
Ext.close();
setBatchMode(false);
showMessage("Done with "+k+" files and "+n+" series!");


function getSaveString(pad) {
  str ="";
  getDimensions(x,y,c,z,t);
  if(t > 1)  str+="t_"+IJ.pad(1,pad);
  if(z > 1)  str+="z_"+IJ.pad(1,pad);
  if(c > 1)  str+="c_"+IJ.pad(1,pad);
  
  return str;
}
