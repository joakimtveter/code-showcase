# Code showcase

I have found some code snippets from my projects that i think demonstrates my current skill level.

## API Call function

This is how I set up my API calls in my school projects. By creating these functions I can easily make API calls with all query paremeter options in a optional options object.
I make one of these for each endpoint in the API.

### Basic Reusable get function with access token

```js
import { showToast, getAccessToken } from '../utils.js';
import { BASE_URL } from '../client.js';

/**
 * Generic function to get data from API, includes access token in header and error handeling.
 * @param {string} endpoint - API endpoint including query parameters and leadning slash
 * @returns {Promise<object[]>}
 */
export async function get(endpoint) {
    const token = getAccessToken();
    try {
        const response = await fetch(BASE_URL + endpoint, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
                Authorization: `Bearer ${token}`,
            },
        });
        if (!response.ok) {
            const error = await response.json();
            throw new Error(`${error.statusCode} ${error.status} - ${error.errors[0].message}`);
        }
        const data = await response.json();
        return data;
    } catch (error) {
        console.error(error);
        showToast(error, 'error');
    }
}
```

### Example API call that gives autocomplete suggestions

```js
import { get } from '../get';

/**
 * Get listings from the API
 * @param {Object} options - Options for the query
 * @param {string} options.sort - The field to sort by
 * @param {string} options.sortOrder - The order to sort by
 * @param {number} options.limit - The number of listings to return
 * @param {number} options.offset - The offset to start from
 * @param {string} options.tag - The tag to filter by
 * @param {boolean} options.active - Whether to return only active listings
 * @param {boolean} options.seller - Whether to add seller object to listing
 * @param {boolean} options.bids - Whether to add bids array to listing
 * @returns {Promise<Object[]>} - A promise that resolves to an array of listings
 */
export function getListings(options = {}) {
    let queryParams = '';
    const parameters = [];

    if (options.sort) parameters.push(`sort=${options.sort}`);
    if (options.sortOrder) parameters.push(`sortOrder=${options.sortOrder}`);
    if (options.limit) parameters.push(`limit=${options.limit}`);
    if (options.offset) parameters.push(`offset=${options.offset}`);
    if (options.tag) parameters.push(`_tag=${options.tag}`);
    if (options.active) parameters.push(`_active=${options.active}`);
    if (options.seller) parameters.push(`_seller=${options.seller}`);
    if (options.bids) parameters.push(`_bids=${options.bids}`);

    if (parameters.length > 0) queryParams = '?' + parameters.join('&');
    return get(`/listings${queryParams}`);
}
```

## Mulitple choice feedback question with exclusive answers

This code was part of a feedback form prototype for a client. The client is moving forward with the project and the code will be used in the final product.
The stack was Vite, React, React Router, Formik, and Material UI. The prodotype used Firebase as a backend to store the form submissons. The final product will use an excisting .NET platform to generate questions and store the result.

### Custom Checkbox component

```js
import { useField } from 'formik';
import { Checkbox, FormControlLabel, Grid, Radio } from '@mui/material';
import CheckBoxIcon from '@mui/icons-material/CheckBoxOutlineBlank';
import CheckBoxCheckedIcon from '@mui/icons-material/CheckBox';
import RadioIcon from '@mui/icons-material/RadioButtonUnchecked';
import RadioCheckedIcon from '@mui/icons-material/CheckCircle';

export default function FormikCheckbox({ label, exclusive, exclusiveOptions, ...props }) {
    const [field, , helpers] = useField({ ...props, type: 'checkbox' });
    const { value } = field;

    function handleChange(option) {
        // If the option is exclusive, set the value to be an array with only that option
        if (exclusiveOptions.includes(option)) {
            const newValue = [option];
            helpers.setValue(newValue);
            return;
        }

        // If the option is not exclusive, but the value already includes an exclusive option, set the value to be an array with only that option
        if (exclusiveOptions.some((r) => value.includes(r))) {
            const newValue = [option];
            helpers.setValue(newValue);
            return;
        }

        // If the option is not exclusive and the value does not include an exclusive option, add or remove the option from the value
        if (value.includes(option)) {
            const filteredValues = value.filter((val) => val !== option);
            helpers.setValue(filteredValues);
        } else {
            helpers.setValue([...value, option]);
        }
    }

    return (
        <Grid item xs={6}>
            <FormControlLabel
                {...field}
                control={
                    <Checkbox
                        onChange={() => handleChange(label.props.label)}
                        checked={field.value.includes(label.props.label)}
                        icon={exclusive ? <RadioIcon /> : <CheckBoxIcon />}
                        checkedIcon={exclusive ? <RadioCheckedIcon /> : <CheckBoxCheckedIcon />}
                        sx={{
                            color: 'text.green',
                            '&.Mui-checked': {
                                color: 'text.green',
                            },
                        }}
                    />
                }
                label={label}
            />
        </Grid>
    );
}
```

### Component in use

```js
import React from 'react';
import { FormGroup, Grid } from '@mui/material';
import QuestionText from './partials/QuestionText';
import FormikCheckbox from '../formik/formikCheckbox';
import CustomLabel from './partials/CustomLabel';

export default function PickMultipleQuestion(props) {
    const { name, number, question, helptext, preText, options } = props;
    const exclusiveOptions = options.filter((option) => option.exclusive).map((option) => option.label);

    return (
        <>
            <QuestionText number={number} name={name} question={question} helptext={helptext} preText={preText}>
                <FormGroup aria-labelledby={`q-${name}`} sx={{ paddingInline: 2 }}>
                    <Grid container rowGap={3}>
                        {options.map((option) => (
                            <FormikCheckbox
                                key={option.label}
                                name={name}
                                label={<CustomLabel {...option} />}
                                sx={{ marginBlockEnd: 1 }}
                                exclusive={!!option.exclusive}
                                exclusiveOptions={exclusiveOptions}
                            />
                        ))}
                    </Grid>
                </FormGroup>
            </QuestionText>
        </>
    );
}
```
