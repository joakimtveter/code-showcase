# Code showcase

I have found some code snippets from my projects that i think demonstrates my current skill level.

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
