# Component-library-with-typescript

### Shortcut list:
1. [Window Width Provider](#window-width)
2. [Dynamic Heading](#dynamic-heading)
3. [Next js Dynamic Title (Meta Data)](#dynamic-heading-next)
4. [Jod editor](#jod-editor)
5. [Translator](#translator)
6. [Custom Buttom](#custom-button)
7. [Custom Input](#custom-input)
8. [Custom Select](#custom-select)

<div id="window-width">

## 1. Window width context provider:

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

</div>

<div id="dynamic-heading">
  
## 2. Dynamic heading:

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
</div>

<div id="dynamic-heading-next">
  
# Dynamic meta data for Next.js

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
</div>

<div id="jod-editor">

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
</div>

<div id="quil-editor">

## Quil Editor Setup 

### Install React Quill Editor
```bash
npm i react-quill-new
```

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
<form>
<QuillEditor
value={form.getFieldValue("experience")}
onChange={(content) => form.setFieldsValue({ experience: content })}
placeholder="Write here your experience"
/>
</form>
```

</div>

<div id="translator">

## Next Translator Provider
### Provider
```bash
/* eslint-disable @typescript-eslint/no-explicit-any */
"use client";

import { Select } from "antd";
import { Option } from "antd/es/mentions";
import { useEffect, useState } from "react";
// import { RiGlobalLine } from "react-icons/ri";

declare global {
  interface Window {
    googleTranslateElementInit?: () => void;
    google?: any;
  }
}

function GoogleTranslateProvider({ children }: { children: React.ReactNode }) {
  const [isInitialized, setIsInitialized] = useState(false);

  useEffect(() => {
    const addGoogleTranslateScript = () => {
      if (!document.querySelector("#google-translate-script")) {
        const script = document.createElement("script");
        script.id = "google-translate-script";
        script.src =
          "https://translate.google.com/translate_a/element.js?cb=googleTranslateElementInit";
        script.async = true;
        document.body.appendChild(script);
      }

      window.googleTranslateElementInit = () => {
        if (window.google && !isInitialized) {
          new window.google.translate.TranslateElement(
            {
              pageLanguage: "en",
              includedLanguages: "en,fr,iu,es,de,ar,pt,hi,bn",
              layout:
                window.google.translate.TranslateElement.InlineLayout
                  .HORIZONTAL,
            },
            "google_translate_element"
          );
          setIsInitialized(true);
        }
      };
    };

    addGoogleTranslateScript();
  }, [isInitialized]);

  return (
    <>
      <div id="google_translate_element" className="hidden"></div>
      {children}
    </>
  );
}

export default GoogleTranslateProvider;

export function LanguageSwitcher() {
  const [selectedLanguage, setSelectedLanguage] = useState("en");

  useEffect(() => {
    // Fetch stored language from localStorage and set default language if not available
    const storedLang = localStorage.getItem("selectedLanguage");
    if (storedLang) {
      setSelectedLanguage(storedLang);
    } else {
      setSelectedLanguage("en"); // Default to English if no language is stored
    }

    // Set the initial googtrans cookie based on stored or default language
    if (storedLang === "en" || !storedLang) {
      document.cookie =
        "googtrans=/en/en; expires=Thu, 31 Dec 2099 23:59:59 UTC; path=/";
    } else {
      document.cookie = `googtrans=/en/${storedLang}; expires=Thu, 31 Dec 2099 23:59:59 UTC; path=/`;
    }
  }, [selectedLanguage]);

  const handleChange = (newLang: string) => {
    if (!newLang) return;

    if (newLang) {
      setSelectedLanguage(newLang);
      localStorage.setItem("selectedLanguage", newLang);

      if (newLang === "en") {
        // Set the googtrans cookie for English
        document.cookie =
          "googtrans=/en/en; expires=Thu, 31 Dec 2099 23:59:59 UTC; path=/";
      } else {
        document.cookie = `googtrans=/en/${newLang}; expires=Thu, 31 Dec 2099 23:59:59 UTC; path=/`;
      }

      // Manually trigger the language change in Google Translate
      const select = document.querySelector(
        ".goog-te-combo"
      ) as HTMLSelectElement;
      if (select) {
        select.value = newLang;
        select.dispatchEvent(new Event("change"));
      }
    }
  };

  const languageOptions: { [key: string]: string } = {
    en: "English",
    fr: "French",
    // iu: "Inuktut (Syllabics)",
    es: "Spanish",
    de: "German",
    ar: "Arabic",
    pt: "Portuguese",
    hi: "Hindi",
    bn: "Bangla",
  };

  return (
    <div className="flex items-center gap-2">
      {/* <span className="hidden text-gray-700 font-medium md:flex justify-center items-center gap-1">
        <RiGlobalLine />
      </span> */}
      <Select
        value={selectedLanguage || undefined}
        onChange={handleChange}
        placeholder="Select a language"
        className="font-bold text-black/70"
        style={{ width: 200, borderRadius: 9999 }} // Tailwind rounded-full equivalent
      >
        {Object.entries(languageOptions).map(([code, name]) => (
          <Option key={code} value={code}>
            {name}
          </Option>
        ))}
      </Select>
    </div>
  );
}

```

### Global CSS file
```bash
body {
  top: 0 !important;
  position: static !important;
}

body .skiptranslate {
  display: none !important;
}

.goog-te-banner-frame.skiptranslate {
  display: none !important;
}

.VIpgJd-ZVi9od-ORHb-OEVmcd.skiptranslate {
  visibility: hidden !important;
  height: 0 !important;
  position: absolute !important;
  top: 0 !important;
}

```
  
</div>

<div id="custom-button">

### Custom Button

```bash
"use client";

import type React from "react";

interface MyButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: "primary" | "secondary" | "outline";
  size?: "sm" | "md" | "lg";
  className?: string;
}

export const Button: React.FC<MyButtonProps> = ({
  children,
  onClick,
  variant = "primary",
  size = "md",
  className = "",
}) => {
  // Define base styles
  const baseStyles =
    "rounded-md cursor-pointer font-medium transition-all duration-200 flex items-center justify-center";

  // Define variant styles
  const variantStyles = {
    primary:
      "bg-accent text-white px-6 py-3 rounded-md font-semibold hover:bg-accent-light",
    secondary: "bg-gray-800 text-white hover:bg-gray-900",
    outline:
      "border-2 border-gray-300 text-secondary-text hover:bg-secondary font-semibold hover:text-secondary-text-light",
  };

  // Define size styles
  const sizeStyles = {
    sm: "text-xs px-3 py-1",
    md: "text-sm px-4 py-2",
    lg: "text-base px-6 py-3",
  };

  return (
    <button
      onClick={onClick}
      className={`${baseStyles} ${variantStyles[variant]} ${sizeStyles[size]} ${className}`}
    >
      {children}
    </button>
  );
};

```

</div>

<div id="custom-input">

### Custom Input

```bash
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

// Define input variants
const inputVariants = cva(
  `flex w-full rounded-md border-0 bg-secondary px-3 py-3 text-sm 
   ring-offset-background file:border-0 file:bg-transparent file:text-sm 
   file:font-medium placeholder:text-muted-foreground focus-visible:outline-none 
   focus-visible:ring-0 disabled:cursor-not-allowed disabled:opacity-50`,
  {
    variants: {
      variant: {
        default: "",
        ghost: "bg-transparent border-none shadow-none",
        underline: "border-b rounded-none bg-transparent px-0",
      },
      inputSize: {
        sm: "h-8 px-2 text-sm",
        md: "h-10 px-3 text-base",
        lg: "h-12 px-4 text-lg",
      },
    },
    defaultVariants: {
      variant: "default",
      inputSize: "lg",
    },
  }
);

export interface InputProps
  extends Omit<React.InputHTMLAttributes<HTMLInputElement>, "size">,
    VariantProps<typeof inputVariants> {
  label?: string;
  error?: string;
}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  (
    { className, type = "text", label, error, variant, inputSize, ...props },
    ref
  ) => {
    return (
      <div className="w-full">
        {label && (
          <label className="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">
            {label}
          </label>
        )}
        <input
          type={type}
          className={cn(
            inputVariants({ variant, inputSize }),
            error && "border-red-500 focus-visible:ring-red-500",
            className
          )}
          ref={ref}
          {...props}
        />
        {error && <p className="mt-1 text-sm text-red-500">{error}</p>}
      </div>
    );
  }
);

Input.displayName = "Input";

export { Input, inputVariants };

```

</div>

<div id="custom-select">

### Custom Select

```bash
'use client';

import { useState, useRef, useEffect } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { ChevronDown, Check } from 'lucide-react';

type Option = {
  label: string;
  value: string;
};

type CustomSelectProps = {
  options: Option[];
  placeholder?: string;
  label?: string;
  onChange?: (value: string) => void;
  defaultValue?: string;
};

export default function CustomSelect({
  options,
  placeholder = 'Select...',
  label,
  onChange,
  defaultValue,
}: CustomSelectProps) {
  const [isOpen, setIsOpen] = useState(false);
  const [selected, setSelected] = useState<Option | null>(
    () => options.find((o) => o.value === defaultValue) ?? null
  );

  const selectRef = useRef<HTMLDivElement>(null);

  const handleSelect = (option: Option) => {
    setSelected(option);
    setIsOpen(false);
    onChange?.(option.value);
  };

  // Close dropdown on outside click
  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (selectRef.current && !selectRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);

  return (
    <div className="w-full max-w-sm space-y-2" ref={selectRef}>
      {label && <label className="text-sm font-medium text-muted-foreground">{label}</label>}

      <button
        onClick={() => setIsOpen((prev) => !prev)}
        className="flex w-full items-center justify-between rounded-md border border-input bg-background px-3 py-2 text-sm shadow-sm transition-all focus:outline-none focus:ring-2 focus:ring-ring"
      >
        <span>{selected?.label ?? placeholder}</span>
        <ChevronDown className="w-4 h-4 opacity-60" />
      </button>

      <AnimatePresence>
        {isOpen && (
          <motion.ul
            initial={{ opacity: 0, y: -8, scale: 0.95 }}
            animate={{ opacity: 1, y: 0, scale: 1 }}
            exit={{ opacity: 0, y: -8, scale: 0.95 }}
            transition={{ duration: 0.15 }}
            className="absolute z-50 mt-2 w-full max-w-sm rounded-md border bg-white shadow-xl"
          >
            {options?.length > 0 ? (
              options.map((option) => (
                <li
                  key={option.value}
                  onClick={() => handleSelect(option)}
                  className="flex cursor-pointer items-center justify-between px-3 py-2 text-sm hover:bg-accent hover:text-accent-foreground"
                >
                  {option.label}
                  {selected?.value === option.value && <Check className="w-4 h-4 text-green-500" />}
                </li>
              ))
            ) : (
              <li className="px-3 py-2 text-sm text-muted-foreground">No options available</li>
            )}
          </motion.ul>
        )}
      </AnimatePresence>
    </div>
  );
}

```

</div>

