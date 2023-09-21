# Code showcase

I have found some code snippets from my projects that I think demonstrates my current skill level.

## Vanilla JS Utility functions

### Create a HTML element

I used this extensivly in my early school projects. It really helps to keep the code clean when creating HTML elements in JavaScript.

```js
/**
 * Creates a HTML element with the given tag, class name and content.
 * @param {string} tag - HTML element tag name of the element
 * @param {string} [className] - Class name of the element
 * @param {string} [content] - Inner text of the element
 * @param {object} [atributtes] - Object with the element's attributes
 * @returns {HTMLElement} - Returns the created HTML element
 */
export function createHtmlElement(tag, className = null, content = null, atributtes = {}) {
    const element = document.createElement(tag);
    if (className) element.className = className;
    if (content) element.innerText = content;
    for (const key in atributtes) {
        element.setAttribute(key, atributtes[key]);
    }
    return element;
}
```

### Update the query string in the URL without reloading the page

```js
/**
 * Set mulitple query parameters and values in the URL query string without reloading the page
 * @param {object} [parameters] - object with key value pairs of parameters and values, no parameters will remove all parameters.
 * @returns {void} Sets the value of a parameter in the URL query string
 */
export function setUrlParametersWithoutReload(parameters = {}) {
    const currentPath = window.location.pathname;
    if (Object.keys(parameters).length === 0) {
        window.history.pushState({}, '', currentPath);
    } else {
        let queryparams = [];
        Object.entries(parameters).forEach(([key, value]) => {
            queryparams.push(`${key}=${value}`);
        });

        window.history.pushState({}, '', currentPath + '?' + queryparams.join('&'));
    }
}
```

### Get the value of a query parameter from the URL

```js
/**
 * Get the value of a parameter from the URL query string
 * @param {string} parameter Url parameter to get the value from
 * @returns {string} Value of the parameter
 */
export function getValueFromURLParameter(parameter) {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(parameter);
}
```

### Format date for vanilla HTML date picker

```js
/**
 * Formats a date object to a string that can be used in a date input field
 * @param {object} date - a date object
 * @returns - a string formatted for the datepicker
 */

export function formatDateforDatepicker(date) {
    const year = date.getFullYear();
    const month = ('0' + (date.getMonth() + 1)).slice(-2);
    const day = ('0' + date.getDate()).slice(-2);
    const hours = ('0' + date.getHours()).slice(-2);
    const minutes = ('0' + date.getMinutes()).slice(-2);
    return `${year}-${month}-${day}T${hours}:${minutes}`;
}
```

## API Call function

This is how I set up my API calls in my school projects. By creating these functions I can easily make API calls with all query paremeter options in a optional options object.
I make one of these for each endpoint in the API.

### Basic Reusable get function with access token

THis lets me make athenticated calls without having to get the access token every time.
This example is from a social media app that required authentication to get data from the API.

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

In my next school project I will use TypeScript, but since this is from the JavaScript 2 course I documented the function with JSDoc.
In TypeScript i could set the sort order type to `asc | desc` to get even better autocompletion.

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

## Mulitple choice survey question with exclusive answers

This code was part of a feedback survey prototype for a client. The client is moving forward with the project and the code will be used in the final product.
The stack was Vite, React, React Router, Formik, and Material UI. The prodotype used Firebase as a backend to store the form submissons. The final product will use an excisting dotNET platform to generate questions and store the results.

### Custom Checkbox component

The component is wraped with Formik useField hook to get access to the formik state and helpers.
This is the best way I have found to make Formik and Material UI work together.

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

This is how the question data is structured.

```js
const survey = [
    {
        key: 'experience',
        type: 'checkbox',
        initialValue: [],
        preText: 'Fortell litt om din opplevelse.',
        question: 'Var dette en opplevelse der du brukte...',
        helptext: 'Du kan krysse av på flere',
        options: [
            { icon: 'head', label: 'Hodet', helptext: 'for å tenke og undre meg' },
            { icon: 'eyes', label: 'Øynene', helptext: 'til å observere' },
            { icon: 'ears', label: 'Hørsel', helptext: 'for å lytte' },
            { icon: 'feelings', label: 'Følelsene', helptext: 'til å leve meg inn' },
            { icon: 'body', label: 'Hele kroppen', helptext: 'til å bevege meg' },
            { icon: 'voice', label: 'Stemmen', helptext: 'til å uttrykke meg' },
            { icon: 'hands', label: 'Hendene', helptext: 'til å lage ting selv' },
            { icon: 'laugh', label: 'Lattermusklene', helptext: 'til å le' },
            { icon: 'imagination', label: 'Fantasien', helptext: 'til å drømme med' },
            { icon: 'none', label: 'Ingen av delene passer', exclusive: true },
        ],
    },
];
```

And this is how the component in use.

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
