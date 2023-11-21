# Setting Up Netlify Forms with TypeScript

---

This guide will walk you through the process of setting up a Netlify form in a TypeScript project escpecially if you are working with static site generators like Docusaurus. We'll be using a feedback form as an example.

## Project Structure

Here's the folder structure we'll be working with:
```Folder
.
├── src
│   ├── components
│   │   ├── Feedback
│   │   │   ├── FeedbackButton.tsx
│   │   │   └── FeedbackForm.tsx
│   │   └── ...
│   └── ...
└── static
    └── hidden-forms.html
```


## Creating the Form Component

First, let's create the form component. This is where users will input their feedback.

In src/components/Feedback/FeedbackForm.tsx:

```typescript
import React, { useState } from 'react';
import { toast } from 'react-toastify';
import 'react-toastify/dist/ReactToastify.css';

const FeedbackForm = () => {
    const [isOpen, setIsOpen] = useState(false);
    const [feedback, setFeedback] = useState({
        helpful: '',
        firstName: '',
        lastName: '',
        email: '',
        additionalFeedback: '',
        url: window.location.href,
    });

    const handleChange = (e) => {
        setFeedback({ ...feedback, [e.target.name]: e.target.value });
    };

    const handleSubmit = (e) => {
        e.preventDefault();

        const myForm = e.target;
        const formData = new FormData(myForm);
        let formDataObj = {};

        for (let pair of formData.entries()) {
            formDataObj[pair[0]] = pair[1] as string;
        }

        fetch("/", {
            method: "POST",
            headers: { "Content-Type": "application/x-www-form-urlencoded" },
            body: new URLSearchParams(formDataObj).toString(),
        })
            .then(() => {
                toast.success("Your form has been submitted successfully!");
                setIsOpen(false); // Close the form
            })
            .catch((error) => alert(error));
    };

    return (
        <div style={{ position: 'fixed', right: 0, top: '50%', transform: 'translateY(-50%)' }}>
            <button style={{ position: 'fixed', right: 0, top: '50%', transform: 'translateY(-50%) rotate(-90deg)' }} onClick={() => setIsOpen(!isOpen)}>Feedback</button>
            {isOpen && (
                <div style={{ position: 'fixed', right: 0, top: 0, width: '300px', backgroundColor: '#fff', padding: '20px', overflow: 'auto' }}>
                    <form onSubmit={handleSubmit} method="post" name="FeedbackForm" data-netlify="true">
                        <input type="hidden" name="form-name" value="FeedbackForm" />
                        <label>
                            Was this information helpful?
                            <select name="helpful" onChange={handleChange}>
                                <option value="">Select...</option>
                                <option value="yes">Yes</option>
                                <option value="no">No</option>
                            </select>
                        </label>
                        <label>
                            First Name
                            <input type="text" name="firstName" onChange={handleChange} />
                        </label>
                        <label>
                            Last Name
                            <input type="text" name="lastName" onChange={handleChange} />
                        </label>
                        <label>
                            Email
                            <input type="email" name="email" onChange={handleChange} />
                        </label>
                        <label>
                            Additional Feedback
                            <textarea name="additionalFeedback" onChange={handleChange} />
                        </label>
                        <input type="hidden" name="url" value={window.location.href} />
                        <button type="submit">Submit</button>
                    </form>
                </div>
            )}
            <div style={{ display: 'none' }}>
                <form name="FeedbackForm" data-netlify="true">
                    <input type="hidden" name="helpful" />
                    <input type="hidden" name="firstName" />
                    <input type="hidden" name="lastName" />
                    <input type="hidden" name="email" />
                    <input type="hidden" name="additionalFeedback" />
                    <input type="hidden" name="url" />
                </form>
            </div>
        </div>
    );
};

export default FeedbackForm;

 ```


This component includes a form with several fields: helpful, firstName, lastName, email, additionalFeedback, and url. The form has data-netlify="true" attribute, which tells Netlify to handle the form submission.

## Creating the Button Component

Next, let's create a button that will toggle the visibility of the form.

In src/components/Feedback/FeedbackButton.tsx:
```typescript
import React from 'react';
import FeedbackForm from './FeedbackForm';

interface FeedbackButtonProps {
    className?: string;
}

const FeedbackButton: React.FC<FeedbackButtonProps> = ({ className }) => {
    const [isFeedbackFormVisible, setIsFeedbackFormVisible] = React.useState<boolean>(false);

    const toggleFeedbackForm = () => {
        setIsFeedbackFormVisible((prevState) => !prevState);
    };

    return (
        <>
            <button className={`feedback-button ${className}`} onClick={toggleFeedbackForm}>
                Feedback
            </button>

            {isFeedbackFormVisible && (
                <div className="feedback-form">
                    <FeedbackForm />
                </div>
            )}
        </>
    );
};

export default FeedbackButton;
```

This component includes a button that toggles the visibility of the FeedbackForm component when clicked.

## Creating the Hidden Form

Netlify forms work by parsing static HTML forms at build time. However, when using React or other JavaScript frameworks, forms are typically created dynamically in the browser, and Netlify can't see them at build time. To get around this, we create a hidden form in a static HTML file that Netlify can parse.

In static/hidden-forms.html:
```html
<!DOCTYPE html>
<html>

<body>
    <form name="FeedbackForm" data-netlify="true" netlify-honeypot="bot-field" hidden>
        <input type="hidden" name="helpful" />
        <input type="text" name="firstName" />
        <input type="text" name="lastName" />
        <input type="email" name="email" />
        <textarea name="additionalFeedback"></textarea>
        <input type="hidden" name="url" />
    </form>
</body>

</html>
```

This hidden form has the same name attribute and input name attributes as the form in our FeedbackForm component. This is necessary for Netlify to correctly process form submissions.

## Layout Component

The Layout component is a crucial part of your application as it wraps the main content of your pages and can be used to include common elements across different pages such as headers, footers, navigation menus, or sidebars.

In our project, we have a Layout component located at src/theme/Layout/index.js. Here's what it looks like:
```javascript
import React from 'react';
import OriginalLayout from '@theme-original/Layout';
import FeedbackForm from '../../components/Feedback/FeedbackForm';
import { ToastContainer } from 'react-toastify';

const Layout = (props) => (
  <OriginalLayout {...props}>
    {props.children}
    <FeedbackForm />
    <ToastContainer />
  </OriginalLayout>
);

export default Layout;
```

In this component, we're importing and using an OriginalLayout component from the original theme. We're also importing a FeedbackForm component and a ToastContainer from 'react-toastify' for displaying notifications.

The Layout component receives props and passes them to the OriginalLayout component using the spread operator (...props). This means that any props we pass to Layout will also be passed to OriginalLayout.

Inside the OriginalLayout component, we render props.children, which represents the main content of each page, a FeedbackForm for collecting user feedback, and a ToastContainer for displaying notifications.

By wrapping our page content with this Layout component, we ensure that each page in our application includes a feedback form and a container for notifications

## Conclusion

That's it! You've now set up a Netlify form in a TypeScript project. When a user submits the form, Netlify will handle the form submission and you can view the submissions in your Netlify dashboard.
