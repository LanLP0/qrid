# QRID

App for generating specific QR codes that serve to identify people

## Live in GitHub Pages 
https://lanlp0.github.io/qrid/

## Specifications
| Variant | Required Fields                    |
| :------ | :--------------------------------- |
| v1      | Name, PhoneFirst4, Salt, PhoneHash |

Every QRID QR must contains the following content as plain text
```ini
# Must have on the first line
qrid=v1

# Example comment (only full line allowed)

# Category
[Main]
# All required values go here
# For example,
Name=Trinh Thai Hong
Phone=023342091625
Salt=azRm@igLaX
PhoneHash=da494a6d93919c0761e3eb00c22175106e9b1e8b8aad3fb65f9414da75591872

# Categories can be used to add additional informations
# Required informations are specified by the varient number
```

### Further Specifications

- v1  
PhoneHash field is calculated by `sha256(phone + salt)`