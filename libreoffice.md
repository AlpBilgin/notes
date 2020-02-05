# Libreoffice

## calc

### how do I remove string formatting '?

The apostrophe isn't really part of the cell content, it just signifies that the cell is formatted as text. To reenter all the data as number:
1. Highlight all of the cells and use Format -> Cells to change the cell format to an appropriate number format.
2. With all of the cells still selected, go to the menu Edit -> Find & Replace
3. In the Search For box enter .* (period asterisk)
4. In the Replace with box enter &
5. Select More Options and check Current Selection Only and Regular Expressions
6. Click Replace All

The .* "means zero or more of any character" and & means "whatever was found". These are regular expressions which are explained in the help section
