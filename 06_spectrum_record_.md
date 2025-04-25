# Chapter 6: Spectrum Record

In [Chapter 5: Data Retrieval (`client.retrieve`, `client.retrieve_by_specid`)](05_data_retrieval___client_retrieve____client_retrieve_by_specid___.md), we learned how to ask SPARCL to fetch the *full data* for spectra using their unique identifiers (`sparcl_id` or `specid`). We saw that methods like `client.retrieve()` return a `Retrieved` object, like `results_I` in our examples.

```python
# From Chapter 5:
# ids_I = found_I.ids
# inc = ['sparcl_id', 'specid', 'data_release', 'redshift', 'flux',
#        'wavelength', 'model', 'ivar', 'mask', 'spectype', 'ra', 'dec']
# results_I = client.retrieve(uuid_list=ids_I, include=inc)

# We know the data is in results_I.records
# For example, the first record:
first_retrieved_record = results_I.records[0]
print(first_retrieved_record)
```

The output might look something like this (arrays shortened for display):
```text
(_dr=DESI-DR1, sparcl_id=000003e8-8a05-11ef-a71d-525400f334e1, specid=39633285986387037, data_release=DESI-DR1, redshift=0.782049, flux=array([-0.338,  0.032,  4.36 , ...,  6.68 ,  5.72 ,  5.72 ], dtype=float32), wavelength=array([3600. , 3600.8, 3601.6, ..., 9822.4, 9823.2, 9824. ], dtype=float32), model=array([-0.193, -0.139, -0.084, ...,  6.69 ,  6.69 ,  6.69 ], dtype=float32), ivar=array([0.   , 0.   , 0.   , ..., 0.436, 0.379, 0.   ], dtype=float32), mask=array([1024, 1024, 1024, ...,    0,    0, 1024], dtype=uint32), spectype=GALAXY, ra=60.007777, dec=-0.877601)
```

This output shows a mix of things – identifiers, numbers like redshift, coordinates like RA/Dec, and even arrays labeled `flux`, `wavelength`, `model`, `ivar`, and `mask`. What exactly is this `first_retrieved_record` object, and how is all this information organized?

## Motivation: Understanding the Individual "File Card"

We've received our delivery of spectra (the `Retrieved` object). Think of it as a box filled with detailed information cards for each spectrum we requested. We need to understand how to read one of these cards. What information does it contain, and how is it structured?

This individual "card" is what we call a **Spectrum Record**. It's the fundamental package of data and context for a single spectrum returned by a retrieval request.

**Our Goal:** In this chapter, we'll learn about the structure of a `Spectrum Record` object (like the ones inside `results_I.records`). We'll see what key pieces of information it holds – both the actual light measurements and the descriptive details – and how to access them.

## What is a Spectrum Record?

A `Spectrum Record` is the main unit of data you get back when you use `client.retrieve()` or `client.retrieve_by_specid()`. Each record represents **one single astronomical spectrum** and bundles together all the important information associated with it.

Think of it like a detailed **digital file card** for a specific spectrum. This card holds two main types of information:

1.  **Observational Data (The Measurements):** This is the core scientific data – the "light fingerprint" itself.
    *   **`flux`**: An array of numbers representing the measured brightness (or flux density) of the object's light.
    *   **`wavelength`**: A corresponding array of numbers telling you the specific "color" (or wavelength) of light for each `flux` measurement. Together, `flux` and `wavelength` make the spectrum plot.
    *   **`ivar` (Inverse Variance)**: An array giving information about the *uncertainty* or *error* in each `flux` measurement. Higher `ivar` means a more certain (lower error) measurement; lower `ivar` means a less certain (higher error) measurement. It's like a quality score for each data point.
    *   **`mask`**: An array of codes (flags) indicating if there were any known issues with specific data points (e.g., affected by cosmic rays, bad detector pixels). This helps scientists know which points might be unreliable.
    *   **(Optional) `model`**: Sometimes, a best-fit model spectrum is also included, which can be compared to the observed `flux`.

2.  **Metadata (The Context):** This is the descriptive information *about* the spectrum – where it came from, what object it is, etc.
    *   **`sparcl_id`**: The unique SPARCL identifier ([Chapter 4: Identifier (`sparcl_id`, `specid`)](04_identifier___sparcl_id____specid___.md)).
    *   **`specid`**: The original identifier from the survey ([Chapter 4: Identifier (`sparcl_id`, `specid`)](04_identifier___sparcl_id____specid___.md)).
    *   **`ra`, `dec`**: The sky coordinates (Right Ascension and Declination) telling you where the object is located.
    *   **`redshift`**: A measure of how much the object's light has been stretched due to the expansion of the universe (or its motion). Crucial for understanding distances and properties of galaxies and QSOs.
    *   **`spectype`**: The classification of the object (e.g., `GALAXY`, `STAR`, `QSO`).
    *   **`data_release`**: Which data release (e.g., `DESI-DR1`, `SDSS-DR16`) the spectrum belongs to.
    *   ...and potentially other metadata depending on the survey and what you requested using the `include` parameter ([Chapter 7: Field Selection (`include` parameter, `get_all/default_fields`)](07_field_selection___include__parameter___get_all_default_fields___.md)).

Essentially, the `Spectrum Record` combines the **what** (the object type, its location, redshift) with the **how** (the measured light data and its quality).

**Important Note:** Remember the `Record` objects we saw inside the `Found` object from `client.find()` in [Chapter 3: Result Handling (`Found`, `Retrieved` objects)](03_result_handling___found____retrieved__objects__.md)? Those *only* contained metadata. The `Spectrum Record` objects inside a `Retrieved` object (like `results_I`) contain *both* metadata *and* the observational data arrays (`flux`, `wavelength`, `ivar`, etc.).

## Accessing Data within a Spectrum Record

Accessing the information stored in a `Spectrum Record` is very straightforward. As we saw in Chapter 3, the `results_I.records` attribute is a list, and each item in that list is a `Record` object.

```python
# Assume results_I was created in Chapter 5
# results_I = client.retrieve(uuid_list=ids_I, include=inc)

# Get the first record from the list
first_spectrum_record = results_I.records[0]

# What type is it? (Should be sparcl.Results.Record)
print(f"Type of record: {type(first_spectrum_record)}")
```
Output:
```text
Type of record: <class 'sparcl.Results.Record'>
```

To access the individual pieces of data (like `flux`, `wavelength`, `redshift`, etc.) stored within this `Record` object, you use **dot notation**, just like accessing attributes of any Python object.

```python
# Access some metadata
sparcl_id = first_spectrum_record.sparcl_id
spec_id = first_spectrum_record.specid
ra = first_spectrum_record.ra
redshift = first_spectrum_record.redshift
spec_type = first_spectrum_record.spectype

print(f"SPARCL ID: {sparcl_id}")
print(f"Spec ID:   {spec_id}")
print(f"RA:        {ra}")
print(f"Redshift:  {redshift}")
print(f"Spec Type: {spec_type}")

# Access the observational data arrays
flux_data = first_spectrum_record.flux
wavelength_data = first_spectrum_record.wavelength
ivar_data = first_spectrum_record.ivar

# Check the type and shape of the flux array (it's a NumPy array!)
import numpy as np # Need numpy to check type
print(f"\nType of flux data: {type(flux_data)}")
print(f"Shape of flux data: {flux_data.shape}")
# Print the first 5 flux values
print(f"First 5 flux values: {flux_data[:5]}")
```
Output might look like:
```text
SPARCL ID: 000003e8-8a05-11ef-a71d-525400f334e1
Spec ID:   39633285986387037
RA:        60.007777
Redshift:  0.782049
Spec Type: GALAXY

Type of flux data: <class 'numpy.ndarray'>
Shape of flux data: (7781,)
First 5 flux values: [-0.3382929  0.03231603  4.36086    -0.3959809  -0.4103089 ]
```

As you can see, accessing the data is as simple as `record_object.attribute_name`. The attributes `flux`, `wavelength`, `ivar`, `mask`, and `model` (if requested) typically hold NumPy arrays, which are powerful data structures for numerical operations in Python, perfect for analyzing spectra. The other attributes (like `sparcl_id`, `redshift`, `ra`) hold single values (strings, numbers).

## Under the Hood: How Spectrum Records are Structured

We learned in [Chapter 5: Data Retrieval (`client.retrieve`, `client.retrieve_by_specid`)](05_data_retrieval___client_retrieve____client_retrieve_by_specid___.md) that when you call `client.retrieve()`, the SPARCL server sends back a package containing the requested data. The `SparclClient` then receives this package, unpacks it, and creates a `Retrieved` object.

Inside the `Retrieved` object, the `records` attribute is built as a list. For each spectrum successfully retrieved from the server, the client does the following:
1.  **Creates a `Record` Object:** It makes an instance of the `sparcl.Results.Record` class (or a similar internal class).
2.  **Parses Data:** It takes the data for that specific spectrum from the server's response package. This includes both the metadata (like ID, redshift) and the observational data arrays (flux, wavelength, etc.).
3.  **Populates Attributes:** It assigns these pieces of data as attributes to the newly created `Record` object using dot notation (e.g., setting `record.flux = the_flux_array`, `record.redshift = the_redshift_value`). The spectral data arrays are often loaded directly into NumPy `ndarray` objects.

So, the `Record` object acts as a convenient container, making all the different pieces of information for one spectrum easily accessible via simple attribute lookup (like `record.flux`).

```python
# Simplified conceptual view of the Record class (from Chapter 3, but relevant here)
# (Potentially in sparcl/Results.py)

# import numpy as np # Assuming numpy is imported

class Record:
    def __init__(self, data_dict):
        # Store the raw dictionary data
        self._data = data_dict
        
        # Create attributes for easy access
        # For retrieved spectra, data_dict would contain keys like 
        # 'flux', 'wavelength', 'ivar', 'mask', 'model', 'redshift', 'sparcl_id', etc.
        # Values for flux, wavelength, ivar, mask, model would be NumPy arrays.
        for key, value in data_dict.items():
            setattr(self, key, value)
            
    def __repr__(self):
        # Define how the record looks when you print it
        # Avoid printing the full long arrays for flux, wavelength, etc.
        items_list = []
        for k, v in self._data.items():
            is_array = isinstance(v, (list, np.ndarray))
            # Truncate representation of arrays
            value_repr = repr(v[:3]) + '...' if is_array and len(v) > 3 else repr(v)
            if is_array:
                 value_repr = f"array({value_repr}, shape={getattr(v,'shape',len(v))})" # Show shape
            
            # Shorten long strings like sparcl_id if needed for display
            if isinstance(v, str) and len(v) > 30:
                 value_repr = repr(v[:8] + '...' + v[-8:])

            items_list.append(f"{k}={value_repr}")
            
        items = ', '.join(items_list)
        # Limit overall length if needed
        if len(items) > 200:
             items = items[:197] + '...'
             
        return f"({items})"

# When client.retrieve gets data back, it might do something like:
# server_response = # ... get data from server ...
# records_list = []
# for spectrum_data_dict in server_response['records']: 
#     # spectrum_data_dict contains flux array, wavelength array, redshift, etc.
#     record_obj = Record(spectrum_data_dict)
#     records_list.append(record_obj)
# retrieved_object.records = records_list
```

This object-oriented approach (`Record` objects with attributes) makes working with the retrieved data clean and intuitive.

## What's Next?

We've now explored the contents of a `Spectrum Record` – the detailed "file card" for each spectrum returned by `client.retrieve()`. We know it holds both the observational data (like `flux`, `wavelength`, `ivar` as NumPy arrays) and the essential metadata (like `sparcl_id`, `redshift`, `ra`, `dec`). We also know how to access any of these fields using simple dot notation (e.g., `record.wavelength`).

In our examples, we used an `include` list (`inc`) to tell `client.retrieve()` which fields we wanted. What if we only needed the `flux` and `wavelength`? Or what if we wanted absolutely *every* piece of available information for a spectrum? How do we control which fields are included in the `Spectrum Record` objects that get returned?

In the next chapter, we'll learn about [Chapter 7: Field Selection (`include` parameter, `get_all/default_fields`)](07_field_selection___include__parameter___get_all_default_fields___.md) and how to customize the data retrieval process.

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)