# Component-library-with-typescript

### Shortcut list:
[Translator](#translator)

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

