# List, Numpy, and Pandas Cheatsheet

- [List, Numpy, and Pandas Cheatsheet](#list-numpy-and-pandas-cheatsheet)
  - [List](#list)
    - [Indexing](#indexing)
    - [Merging, stacking, joining, and concatenating lists](#merging-stacking-joining-and-concatenating-lists)
    - [Reshaping, transposing, and repeating lists](#reshaping-transposing-and-repeating-lists)
    - [Sorting and reversing lists](#sorting-and-reversing-lists)
  - [Numpy](#numpy)
    - [Create an array](#create-an-array)
    - [Indexing](#indexing-1)
    - [Merging, stacking, joining, and concatenating arrays](#merging-stacking-joining-and-concatenating-arrays)
    - [Reshaping, transposing, and repeating arrays](#reshaping-transposing-and-repeating-arrays)
    - [Assign/ replace values](#assign-replace-values)
    - [Array attributes](#array-attributes)
    - [Math operations](#math-operations)
  - [Pandas](#pandas)
    - [Create a Series or DataFrame](#create-a-series-or-dataframe)
    - [Renaming columns and index](#renaming-columns-and-index)
    - [Indexing and slicing](#indexing-and-slicing)
      - [Series](#series)
      - [DataFrame](#dataframe)
    - [Merging, stacking, joining, and concatenating DataFrames](#merging-stacking-joining-and-concatenating-dataframes)
    - [Other operations](#other-operations)


## List

### Indexing

```python
list[-1] # Last element
list[-2] # Second-to-last element
list[start:end:step]  # Slicing with steps
[x for x in list if x > 5]  # To select elements based on a condition

# To select elements based on a list of indices
indices = [0, 2, 4]
[list[i] for i in indices] 

list.index(element)  # Find the index of an element. If not found, raises ValueError
```

### Merging, stacking, joining, and concatenating lists

```python
list1 + list2 # Stack lists horizontally (i.e. concatenate)
list1.extend(list2) # Concatenate in place
list1 + [list2] # Join lists along a new axis (don't forget the brackets!)
list.append(element) # Append an element to a list
list1.append(list2) # Append a list to another list
list.extend([element1, element2, ...]) # Append multiple elements
list.insert(index, element) # Insert an element at a specific position
list.remove(element) # Remove an element
list.pop(index) # Pop an element
```

### Reshaping, transposing, and repeating lists

```python
list(zip(*list_of_lists)) # Transpose a list of lists

# Repeats a list n times (Don't forget the brackets!)
repeated_list = [list] * n 
# ATTENTION: This creates a list of lists which are references to the same list!
# When you modify one of them, all of them will change.
# For example:
repeated_list = [[]] * 3 # This creates [[], [], []], but all sublists are references to the same list!!
repeated_list[0].append(1)  # This will assign 1 to all sublists!
print(repeated_list)  # Output: [[1], [1], [1]] instead of [[1], [], []]

# Repeat a list n times without creating references to the same list
repeated_list = [list for _ in range(n)]  # This creates a new list for each sublist

# Flatten a list of lists
[item for sublist in list for item in sublist] 
```

### Sorting and reversing lists

```python
list.sort() # Sort the list in place
sorted(list) # Return a new sorted list
list.reverse() # Reverse the list in place
reversed(list) # Return an iterator of the list in reverse order
```

## Numpy

- `ndarray`: N-dimensional array object in Numpy
- **ATTENTION**: Unlike Python lists, any changes/assignments to a Numpy array will modify the original array in place! If you want to create a copy of the array, use `array.copy()`.

    ```python
    array1 = np.array([1, 2, 3])
    array2 = array1
    array2[0] = 10  # This will change array1 as well!
    array3 = array1.copy()  # This will create a copy of array1
    array3[0] = 20  # This will not change array1
    ```

- Broadcasting: Numpy can perform operations on arrays of different shapes and sizes, automatically expanding the smaller array to match the shape of the larger one.

### Create an array

```python
# create arrays from lists, tuples, etc.
np.array([1, 2, 3]) # 1D array
np.array([[1, 2], [3, 4]]) # 2D array
np.array([1, 2, 3], dtype=np.float64) # Specify data type

# Create arrays of ones and zeros
np.zeros((3, 3)) # 3x3 array of zeros
np.ones((2, 2)) # 2x2 array of ones
np.eye(3) # 3x3 identity matrix

# Create arrays of random numbers
np.random.rand(2, 2) # 2x2 array of numbers in the range [0, 1)
np.random.randn(2, 2) # random numbers from a normal distribution
# Create arrays of random integers

# Create arrays of random integers
# You can specific the range of integers
np.random.randint(0, 10) # Random integer in the range [0, 10)
np.random.randint(0, 10, 5) # 5 random integers as a 1D array
np.random.randint(0, 10, (2, 2)) # 2x2 array of random integers
np.random.randint(0, 10, size=(2, 2)) # Same as above

```

### Indexing

```python
# Indexing syntax
array[row, column] # a specific element
array[row, :] # a specific row
array[:, column] # a specific column
array[start:end, start:end] # Slicing
array[start:end:step, start:end:step] # # Slicing with steps

# Slicing examples
array[:, 0:10:3] # every 3rd column from from columns 0 to 9
array[::2, ::2] # Every other row and column
array[-1, :] # Last row
array[-2, :] # Second-to-last row

# To select elements based on a list of indices
rows = [0, 2, 4]
array[rows, :] # rows at inxex 0, 2, and 4 

array[array > 5] # To select elements based on a condition
# array > 5 returns a "mask" of True or False values and acts as a filter
```
- Note that Numpy arrays are 0-indexed

### Merging, stacking, joining, and concatenating arrays

```python
np.vstack((array1, array2)) # Stack arrays vertically (i.e. concatenate along the first axis/ rows)
np.hstack((array1, array2)) # Stack arrays horizontally (i.e. concatenate along the second axis/ columns)
np.stack((array1, array2), axis=0) # Stack arrays along a new axis (creates a new dimension)
np.concatenate((array1, array2), axis=0) # Concatenate arrays end to end
np.split(array, indices_or_sections, axis=0) # Split arrays into multiple sub-arrays along the specified axis
```

### Reshaping, transposing, and repeating arrays

```python
array.flatten() # Flatten an array (returns a 1D array)
array.reshape(new_shape) # Reshape an array (e.g. array.reshape(2, 6) to reshape a 1D array of 12 elements into a 2D array with 2 rows and 6 columns)
array.T # Transpose an array (swaps rows and columns)
np.repeat(array, repeats, axis=0) # Repeat an array along the specified axis (e.g. np.repeat(array, 2, axis=0) will repeat each row twice)
np.tile(array, reps) # Tile an array (e.g. np.tile(array, (2, 3)) will repeat the array 2 times along the first axis and 3 times along the second axis)
np.resize(array, new_shape) # Resize an array (e.g. np.resize(array, (2, 6)) will resize the array to 2 rows and 6 columns, filling with repeated values if necessary)
```

### Assign/ replace values

```python
array[row, column] = new_value # Assign a new value to a specific element
array[1:3, :] = 5 # Assign a scalar to a slice
array[1:3, :] = np.array([[1, 2], [3, 4]]) # Assign an array to a slice
```

### Array attributes

```python
array.shape # Returns the shape (rows, columns)
array.size # Returns the total number of elements
array.ndim # Returns the number of dimensions
array.dtype # Returns the data type
```

### Math operations

```python
# Element-wise operations (returns an array of the same shape of the inputs)
array1 + array2 # Element-wise addition
array1 * array2 # Element-wise multiplication

# Matrix multiplication
np.dot(array1, array2) 

# statistics
array.sum(axis=0) # Sum of every column
array.sum(axis=1) # Sum of every row
# axis=0 operates downwards vertically (along the rows).
# axis=1 operates across horizontally (along the columns).
# same syntax for mean, min, max, std
```

## Pandas

- Pandas is built on top of Numpy
- a Pandas dataframe is essentially an `ndarray` with labels
- You can pull out the underlying `ndarray` using the `.values` attribute (e.g. `df.values`)
- Pandas allows for custom index and column labels

### Create a Series or DataFrame

```python
# 1D series with custom index
s = pd.Series([1, 2, 3], index=['a', 'b', 'c'])

# 2D dataframe with custom index and columns
data = np.ones((3, 3))
df = pd.DataFrame(data, columns=['A', 'B', 'C'], index=['a', 'b', 'c'])

# Create a DataFrame from a dictionary of lists
# Column names are the keys of the dictionary
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

# Create a DataFrame with a DatetimeIndex as the index
dates = pd.date_range('2025-01-01', '2025-01-03') # Create a range of dates (i.e. DatetimeIndex)
df = pd.DataFrame([1, 2, 3], index=dates)

# Create a DataFrame from a CSV file
df = pd.read_csv(
    'data.csv', 
    index_col='date', # Use the column 'date' as indices
    parse_dates=True,  # Parse strings as dates
    header=0, # (Default): first row contains column headers
    names=['date', 'X', 'Y'], # sets custom column names
    usecols=['date', 'A', 'B'], # Extracts specific columns (use original column names here)
    na_values=['nan']
)
# header=None: The CSV has no header row. Pandas will assign numerical names (0, 1, 2...)
```

### Renaming columns and index

```python

# Using .rename()

# Change specific columns
df = df.rename(columns={'old_A': 'new_A', 'old_B': 'new_B'})
# Rename specific values inside a numerical or string index
df = df.rename(index={'2026-01-01': 'New Year Day'})
# Change the index name (the label of the index axis)
df = df.rename_axis('new_index_label')

#---

# Overwrite all column names at once
df.columns = ['date', 'Metric_X', 'Metric_Y']

# Overwrite the actual index values
df.index = ['row1', 'row2', 'row3'] 

```


### Indexing and slicing

#### Series

```python
s['a'] # Access by label
s[0] # Access by position
s[['a', 'b']] # Access multiple elements
s['a':'b'] # Slicing by label (**inclusive end**)
s[0:2] # Slicing by position (exclusive end, similar to Python lists)

s[s > 1] # Select series elements based on a condition
```

#### DataFrame

```python
# Using .loc and .iloc

df.loc[row_label, column_label] # access a specific element
df.iloc[row_index, column_index] # use implicit index (position)
df.loc['a':'b', 'A':'B'] # Slicing by label (inclusive end)
df.iloc[0:2, 0:2] # Slicing by position (exclusive end)

# You can specific both rows and columns, but by default, if only one label is provided, it is interpreted as a row label
df.loc['a'] # Access a row by label
df.iloc[0] # Access a row by position
df.loc[:, 'A':'C'] # Access a range of columns by label
df.iloc[:, [0:2]] # Access a range of columns by position

# ---

# Accessing elements without .loc or .iloc

# The following approach works only for accessing columns
df['A'] # Access a column by label
df[['A', 'B']] # Access specific columns

# Slicing with conditions
df[df['A'] > 1] # Select df rows based on a condition in a specific column
df[(df['A'] > 1) & (df['B'] < 6)] # Select df rows based on multiple conditions
```

### Merging, stacking, joining, and concatenating DataFrames

```python
# Concatenate DataFrames
pd.concat([df1, df2], axis=0) # Stack DataFrames vertically (concatenate along the first axis)
pd.concat([df1, df2], axis=1) # Stack DataFrames horizontally (concatenate along the second axis)

# Note: A pandas DataFrame is strictly a two-dimensional data structure. axis=2 won't work

# Join DataFrames on index (Missing right values become NaN)

df1.join(df2, how='left') # Left join (default): rows from matching indexes + all left indexes
df1.join(df2, how='right') # Right join
df1.join(df2, how='inner') # Inner join (intersection): keep only matching indexes
df1.join(df2, how='outer') # Outer join (union): keep all rows from all indexes from both dataframes

# Joins three or more DataFrames together simultaneously
df1.join([df2, df3, df4], how='outer')

# Join DataFrames on specific column(s) instead of the index
pd.merge(df1, df2, on='key1') # Merge DataFrames on a specific column
pd.merge(df1, df2, how='left', on='key1') # Left join on a specific column

# If you want to merge two DataFrames on multiple keys:
pd.merge(df1, df2, on=['key1', 'key2']) # e.g. pd.merge(df1, df2, on=['username', 'email'])
# Rows with matching keys will be combined into a single row in the resulting DataFrame.

# If two DataFrames use the same keys, but the column names are different, use this:
pd.merge(df1, df2, left_on='key1', right_on='key2')
# e.g. Both of the two DataFrames, 'customers' and 'orders', have a column for customer ID,
# but 'customers' has 'customer_id' and 'orders' has 'id' as the column name:
pd.merge(customers, orders, left_on='customer_id', right_on='id')
```

### Other operations

```python

# Plot a dataframe:
df.plot()

# Plot and specify title and labels:
import matplotlib.pyplot as plt
df.plot(title='Title', xlabel='Date', ylabel='Price')
plt.show()

# Group by a column and calculate the mean

# Rename columns

# Rename index

# Sorting

```