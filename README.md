# Component-library-with-typescript

## Window width context provider:
```bash
"use client";

import { createContext, useEffect, useState, ReactNode } from "react";

type TContextProvider = {
  windowWidth: number;
};

export const ContextProvider = createContext<TContextProvider | null>(null);
const MyContextProvider = ({ children }: {children: ReactNode}) => {
  const [windowWidth, setWindowWidth] = useState<number | null>(null);

  useEffect(() => {
    if (typeof window !== "undefined") {
      setWindowWidth(window.innerWidth); // Set initial window width

      const handleWindowResize = () => {
        setWindowWidth(window.innerWidth);
      };

      window.addEventListener("resize", handleWindowResize);

      return () => {
        window.removeEventListener("resize", handleWindowResize);
      };
    }
  }, []);

  const infoProvider: TContextProvider | null = windowWidth !== null ? { windowWidth } : null;

  return (
    <ContextProvider.Provider value={infoProvider}>
      {children}
    </ContextProvider.Provider>
  );
};

export default MyContextProvider;
```

## Dynamic heading:
```bash
"use client";

import React, { HtmlHTMLAttributes, useContext } from "react";
import clsx from "clsx";
import { ContextProvider } from "@/lib/MyContextProvider";

interface HeaderProps extends HtmlHTMLAttributes<HTMLHeadElement> {
  color?: "white" | "primary" | "black";
  center?: boolean;
  level?: 1 | 2 | 3 | 4 | 5;
  children?: React.ReactNode;
  className?: string;
}

const Heading: React.FC<HeaderProps> = ({
  color = "black",
  center = false,
  level = 2,
  children,
  className,
  ...rest
}) => {
  const textColor = {
    white: "text-white",
    primary: "text-primary",
    black: "text-header",
  }[color];

  const context = useContext(ContextProvider);
  const windowWidth = context ? context.windowWidth : 0;
  const isSmallScreen = windowWidth < 500;

  const finalLevel = isSmallScreen
    ? level + 1 > 5
      ? 5
      : ((level + 1) as typeof level)
    : (level as typeof level);

  const textSize = {
    1: "text-[75px] font-semibold",
    2: "text-[46px] font-semibold",
    3: "text-[28px] font-semibold",
    4: "text-[17px] font-semibold",
    5: "text-[10px] font-semibold",
  }[finalLevel];

  const Tag: React.ElementType = `h${finalLevel}`;

  return (
    <header className={clsx(center && "text-center")}>
      <Tag className={clsx(textSize, textColor, className)} {...rest}>
        {children}
      </Tag>
    </header>
  );
};

export default Heading;
```

## Jodeditor 

```bash

import React, { useState } from "react";
import JoditEditor from "jodit-react";


const [description, setDescription] = useState<string | undefined>("");

<Form.Item
            label={<div className="text-gray-600 font-medium">Description</div>}
            rules={[{ required: true, message: "Please enter a description" }]}
          >
            <JoditEditor
              value={description}
              onBlur={(newContent) => setDescription(newContent)}
              config={{
                readonly: false,
                placeholder: "Start writing...",
                height: 400,
              }}
            />
</Form.Item>
```

# Dynamic meta data

```bash

export const metadata: Metadata = {
  title: {
    default: "Mach Makers",
    template: "%s |  Mach Makers",
  },
  description: "Hire the Best. Get Hired by the Best",
};

export const metadata: Metadata = {
  title: "Services",
};
```

# Quil Editor Setup

```bash
import React, { forwardRef } from 'react';
import ReactQuill from 'react-quill-new';
import 'react-quill-new/dist/quill.snow.css';

interface QuillEditorProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}

const QuillEditor = forwardRef<ReactQuill, QuillEditorProps>(
  ({ value, onChange, placeholder }, ref) => {
    
    const modules = {
      toolbar: [
        [{ 'header': [1, 2, false] }],
        ['bold', 'italic', 'underline'],
        [{ 'list': 'ordered'}, { 'list': 'bullet' }],
        [{ 'align': [] }],
        ['clean']
      ],
    };

    // CORRECTED formats array - must match exactly what Quill expects
    const formats = [
      'header',
      'bold', 'italic', 'underline',
      'list',  // Just 'list' not 'ordered'/'bullet'
      'align'  // Just 'align' not 'align-left' etc.
    ];

    return (
      <div className="rounded-quill-container">
      <ReactQuill
        ref={ref}
        theme="snow"
        value={value}
        onChange={onChange}
        modules={modules}
        formats={formats}
        placeholder={placeholder}
        style={{ height: '200px', marginBottom: '40px' }}
        className='!rounded-xl'
      />
      </div>
    );
  }
);

QuillEditor.displayName = 'QuillEditor';

export default QuillEditor;
```

## Quil editor use

```bash
<Form.Item name="experience" label="Experience">
<QuillEditor
value={form.getFieldValue("experience")}
onChange={(content) => form.setFieldsValue({ experience: content })}
placeholder="Write here your experience"
/>
</Form.Item>
```
