# Genderize CSV
Crediting: jholtmann - Original creater of this script

## Assignment requirements:
1. Read all columns and find the specific column
2. Add response to the copy of original spreadsheet
3. -OVR

## How To:
1. For requirement 1
    * Read csv file  
    * Extract header, and use regex module to find the specific column (the first name column)  
      ```python
      headers = next(readCSV) # take the first row
      specific_header_index = 0
      for header in headers:
        if re.search("name", header, re.I) != None:
          break
        specific_header_index += 1
      ```
    * Store rest content as a list
2. For requirement 2
    * Generate new headers for the spreadsheet by args.override
    * Write original rows and new response to the output file.
      ```python
      for data in dataset:
        writer.writerow([*rest[0], *list(data.values())[1:]])
        rest.pop(0)
      ```
3. For requirement 3
    * Add argument configuration for '-OVR' to the **ArgumentParser**
    * use args.override to generate new header
    * use args.override to generate the combination of original data and new response
      ```python
      if args.override == True:
        for data in dataset:
            append_response = []
            if data.get('gender'): # null check
                female = 1 if data.get('gender') == 'female' else 0
                append_response = [female, female ^ 1] # xor for 0 <=> 1
            else:
                append_response = [0, 0]
            writer.writerow([*rest[0], *append_response])
            rest.pop(0)
      ```
## Result - Sample
###1. Single column
    * Input file:  
    ![img](https://github.com/schen68/Genderize/blob/main/pictures/in-single-col.png)
    * Output file without ***"-ORV"*** argument:  
    ![img](https://github.com/schen68/Genderize/blob/main/pictures/out-single-col.png)
    * Output file with ***"-ORV"*** argument:  
    ![img](https://github.com/schen68/Genderize/blob/main/pictures/out-single-col-OVR.png)
###2. Multi-column
    * Input file:  
    ![img](https://github.com/schen68/Genderize/blob/main/pictures/in-multi-col.png)
    * Output file without ***"-ORV"*** argument:  
    ![img](https://github.com/schen68/Genderize/blob/main/pictures/out-multi-col.png)
    * Output file with ***"-ORV"*** argument:  
    ![img](https://github.com/schen68/Genderize/blob/main/pictures/out-multi-col_OVR.png)
### Usage:
***Run a test***
```sh
python genderize.py -i test/genderize_test_file.csv -o test/out.csv --catch -OVR
```
***Direction***
```sh
python genderize.py [-h] -i INPUT -o OUTPUT [-k KEY] [-c] [-ns] [-nh]
```

```
optional arguments:
  -h, --help            show this help message and exit
  -k KEY, --key KEY     API key
  -c, --catch           Try to handle errors gracefully
  -a, --auto            Automatically complete gender for identical names
  -nh, --noheader       Input has no header row

required arguments:
  -i INPUT, --input INPUT
                        Input file name
  -o OUTPUT, --output OUTPUT
                        Output file name
```

#### Flag details
- _key_:       specify API key [required for 1000+ names]
- _catch_:     try to gracefully catch and handle errors [recommended]
- _auto_:      only request genders for unique names, then autocomplete the duplicates. May significantly lower API usage (by 50% in big test file, for example) [see "Note" for more info]
- _noheader_:  use if input file has no header row

### Test usage:
```
python genderize.py -i test/test.csv -o test/out.csv --catch
```

### Note:
- API key (https://store.genderize.io) is required when requesting more than 1000 names a month.
- Warning: If an error occurs while executing script with the _auto_ argument, no name-matching will occur. The _.tmp_ file will have all the unique names processed to that point, but the script does not yet support picking up from where it was where an error occured! If an error occurs while processing names without the _auto_ argument, you can just remove the processed names from the input file and continue, this is not possible when using the _auto_ argument.

### Requires:
Required module can be found in "dep" folder or pypi link (see "Dependencies")
```
pip install Genderize-0.1.5-py3-none-any.whl
```
Python 3.* (Known working: 3.6.1)

#### Dependencies:
- https://pypi.python.org/pypi/Genderize / https://github.com/SteelPangolin/genderize

### Features:
- Bulk processing (tested with 600,000+ names)
- Estimates remaining time
- Writes data after processing 10 names so little data is lost if an error occurs 
- Support for genderize.io API key (allows processing of more than 1000 names /mo).

#### "Chunks" explanation:
The Python Genderize client used limits requests to 10 names. To work around this, the code breaks the list of names down into chunks of 10. This approach also has the benefit of preventing data loss in case of a crash/server error as the results are written to the output file every 10 names.
