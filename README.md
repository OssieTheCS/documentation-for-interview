
# Trackunit SDK: Custom Fields

This documentation provides a comprehensive guide on using the Trackunit SDK to create and manage custom fields. The guide covers the following topics:

1.  [The benefits of using custom fields](#the-benefits-of-using-custom-fields)
2.  [Local Development](#local-development)
3.  [Defining a Custom Field](#defining-a-custom-field)
4.  [Changing Custom Field Definitions](#changing-custom-field-definitions)
5.  [Deploying a New Custom Field](#deploying-a-new-custom-field)
6.  [Using a Custom Field](#using-a-custom-field)

## The benefits of using custom fields
Before diving into how to use custom fields, it is helpful to understand what the benefits are and in which  scenarios to consider using the feature. Here are a few examples:


 1. **Fleet Management and Monitoring**: You might want to use custom fields to store additional information about vehicles in your fleet, such as fuel consumption, tire pressure, or custom vehicle attributes. This data can then be used to optimize routes, monitor vehicle health, and schedule preventative maintenance, ultimately reducing operating costs and improving fleet efficiency.

 2. **Driver Performance and Safety**: Custom fields can be added to store information about driver behavior, such as average speed, braking patterns, or safety violations. You can use this data to identify areas for improvement, provide targeted training, and promote safe driving practices within your organization.

4.  **Asset Utilization and Allocation**: Custom fields can be used to store data about asset usage, such as hours of operation, idle time, or custom performance metrics. This data can help your organization optimize asset allocation, identify underutilized equipment, and make data-driven decisions about purchasing or leasing new assets.

5.  **Environmental Impact and Sustainability**: You can use custom fields to store information about the environmental impact of your fleet, such as CO2 emissions, fuel efficiency, or noise levels. This data can be used to measure your organization's environmental footprint, identify opportunities for improvement, and demonstrate compliance with environmental regulations or sustainability goals.

## Local Development
During local development, the app runtime will store all custom fields in local storage in the browser. Developers can easily change custom fields and add new ones without waiting to get a new tile version approved. For a full dev setup guide, refer to the [Local Development](https://github.com/OssieTheCS/documentation-for-interview/local-development) page. To work with custom fields, ensure you have installed the two packages below:

```

npm i @trackunit/iris-app-runtime-core
npm i @trackunit/custom-field-components

```


## Defining a Custom Field

To define a custom field, add a `customFieldDefinitions` array to your Iris app manifest:

    customFieldDefinitions: [
      {
        type: 'STRING',
        entityType: 'ASSET',
        key: 'myKey',
        title: 'String Field',
        uiEditable: true,
        uiVisible: true
      }
    ]

The properties include:

1.  `type`: the custom field type (see [below](#custom-field-types) for all options).

2.  `entityType`: the type of entity a custom field can be used on, such as `ASSET` or `ACCOUNT`. More types might be supported later.

3.  `key`: the programmatic name of the field that cannot be changed after creation. 
<!-- Example below -->

4.  `title`: the UI-visible name of the field.
 
5.  `uiEditable` / `uiVisible`: controls the field's display in the manager UI.

For detailed information on custom field types and their properties, refer to the [Custom Field Types](https://chat.openai.com/chat/f6559d14-7905-444a-8631-4a658682fbee#custom-field-types) section.

**All custom fields defined by a tile will be owned by the tile, and you should consider existing data when changing field definitions.**

## Changing Custom Field Definitions

When changing custom field definitions, you need to follow the general rules and type-specific rules:

### General Rules
-   You cannot change a Custom Field Definition `type` after creation (e.g., changing a string field to a number field).
-   The Definition `Key` must be unique.
-   The `Default Value` must be valid. <!--Elaboration needed-->

### Type Rules
Each field type has special rules:

-   **Drop-down:**
    
	-   If an option is removed from a drop-down list, you must provide a valid replacement value.
	    
	-   Any existing value that contains the deleted value will be changed to the replacement value.
	    
	-   If the replacement value is already selected, a duplicate value will not be created.
	    
	-   When changing a multi-select field to a single-select field, any value with more than one option will be replaced with the default value.
	    
	-   The existing value will be saved for retrieval in the API.
    
-   **Number**
    
	-   Any existing value outside the new min/max range will be replaced with the default value. **If no default value is specified, it will be set to null.**
	    
	-   The existing value will be saved for retrieval in the API. <!-- What does even mean? Can the old value be fetched in the API? -->
    
-   **String**
    
	-   If an existing value is longer than the new maximum length, the value will be truncated.
	    
	-   The existing value will be saved for retrieval in the API.

	- Regex patterns are not supported yet, but will be later. 

## Deploying a New Custom Field

After defining a custom field, it must be validated and published. Trackunit will review the tile, and if approved, the new field definitions will be saved. If existing values do not follow the new definition, they will be replaced with the default value.

<!-- Screenshots or examples of deploying and the validation process -->

> 📌 Any values stored during local dev mode will only be available inside the app and not in other parts of the UI. The validation rules are less strict since we only emulate a custom field’s backend.

## Using a Custom Field

To use a custom field, follow these steps:

1.  Install the Iris App Runtime (if you haven't already):

  ```

npm i @trackunit/iris-app-runtime-core

```

2.  Use the `CustomFieldRuntime` command to retrieve custom fields definitions and values saved to the specified Asset ID:


```ts

import { CustomFieldRuntime, CustomFieldType } from  '@trackunit/iris-app-runtime-core';

  

// Get all custom field values and definitions for your asset ID.

const  myCustomFieldsPromise = CustomFieldRuntime.getCustomFieldsFor({
	type:  'ASSET',
	id:  '<my-asset-id>',
});

```

<!-- Example of what the myCustomFieldsPromise looks like below -->

3.  Save the custom field value by entering a definition `key`, `entity ID`, and a new `value`:


```ts

// Save a custom field value

customFieldRuntime.setCustomFieldsFor(
	{
		type:  'ASSET',
		id:  '<my-asset-id>',
	},
	[
		{
			definitionKey:  'myKey',
			value: 
			{
				type:  CustomFieldType.STRING,
				stringValue:  'My new value',
			}
		}
	]
);

```

### React UI Components

Trackunit provides a React UI Components library to simplify the process of using custom fields in a user interface. To use these components, follow these steps:

1.  Install the `@trackunit/iris-react-ui-components` package  (if you haven't already):


```

npm i @trackunit/custom-field-components

```

2.  See below for a complete example on how to use the component:

```ts

import  React, { useEffect, useState } from  'react';

import {
	CustomFieldRuntime,
	AssetRuntime,
	AssetInfo,
	ValueAndDefinition,
} from  '@trackunit/iris-app-runtime-core';

import {
	Button,
	Card,
	CardBody,
	CardFooter,
	CardHeader,
} from  '@trackunit/react-components';

import { CustomField } from  '@trackunit/custom-field-components';
import { useForm } from  'react-hook-form';

export  const  App: React.FC = () => {
	const [customFields, setCustomFields] = useState<ValueAndDefinition[]>();
	const [asset, setAsset] = useState<AssetInfo>();
	const { register, handleSubmit, formState, setValue } = useForm({
		shouldUnregister:  false,
	});
	
	useEffect(() => {
		(async () => {
			const  updatedAssetInfo = await  AssetRuntime.getAssetInfo();
			setAsset(updatedAssetInfo);
			// Using the CustomFieldRuntime you will be able to obtain all custom fields values and
			// definitions owned by the app.
			const  myCustomFields = await CustomFieldRuntime.getCustomFieldsFor({
			id:  updatedAssetInfo.assetId,
			type:  'ASSET',
			});
			setCustomFields(myCustomFields);
		})();
	}, []);
	// With the component it is possible to render an input component for any custom field by
	// providing a field retrieved from getCustomFieldsFor to the component.
	return (
		<Card>
			<CardHeader 
				heading="Custom Fields"
				subHeading="Showcase for custom fields."
			/>
			<CardBody>
				{customFields?.map((field) => {
				return (
					<CustomField
						field={field}
						key={field.definition.key}
						register={register}
						formState={formState}
						setValue={setValue}
					/>
				);
				})}
			</CardBody>
			<CardFooter>
				<Button
					onClick={handleSubmit((data) => {
						// To save custom field values you need to provide the the entity Id and the new value.
						CustomFieldRuntime.setCustomFieldsFromFormData(
						{
							id: asset?.assetId || '',
							type: 'ASSET',
						},
						data,
						customFields || []
						);
					})}
				>
					Save Changes
				</Button>
			</CardFooter>
		</Card>
	);
};
```
This component automatically handles saving custom field values and updates the provided state when changes are made.

## Custom Field Types:

There are several types of custom fields available, each with their own set of properties and rules.
| Field type   | Description                                                                                                             | Extra properties                                        |
| ------------ | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| BOOLEAN      | Boolean value that can store true or false.                                                                             |                                                         |
| DATE         | Date stored in ISO8601 format.                                                                                          |                                                         |
| DROPDOWN     | A predefined list of options. Supports both single and multi selects.                                                   | `allValues`, `multiSelect`, `dropDownValueReplacements` |
| EMAIL        | Email field. Will be rendered with a mailto link in the UI.                                                             |                                                         |
| NUMBER       | Numeric values                                                                                                          | `minimum`, `maximum`, `isInteger`                       |
| PHONE_NUMBER | Phone number stored in E.164 format. Validated using [Google libphonenumber](https://github.com/google/libphonenumber). |                                                         |
| STRING       | Free text field                                                                                                         | `minimumLength`, `maximumLength`, `pattern`             |
| WEB_ADDRESS  | Web address. Will be shown as a link to open a new window in the UI.                                                    |                                                         | 

# Type specific properties
| Property                    | Field type | Required | Description                                                                                             |
| --------------------------- | ---------- | -------- | ------------------------------------------------------------------------------------------------------- |
| `maximum`                   | NUMBER     | No       | Maximum numeric value                                                                                   |
| `minimum`                   | NUMBER     | No       | Minimum numeric value                                                                                   |
| `isInteger`                 | NUMBER     | No       | Disallow decimal values                                                                                 |
| `maximumLength`             | STRING     | No       | Maximum length of the text                                                                              |
| `minimumLength`             | STRING     | No       | Minimum length of the text                                                                              |
| `pattern`                   | STRING     | No       | The allowed regular expression. Syntax is documented [here](https://github.com/google/re2/wiki/Syntax). |
| `allValues`                 | DROPDOWN   | Yes      | All allowed values                                                                                      |
| `multiSelect`               | DROPDOWN   | No       | Allow multiple options to be selected. Default `false`                                                  |
| `dropDownValueReplacements` | DROPDOWN   | No       | Map from old values no longer allowed to new values. Used for updating existing data                    |

> 📌 All custom fields defined by an app will be owned by the app, and you should consider existing data when changing field definitions.
