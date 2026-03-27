# collegelandingpage

"use client";

import { useEffect, useState } from "react";
import Script from "next/script";
import Image from "next/image";
import { usePathname } from "next/navigation";

declare global {
  interface Window {
    googleTranslateElementInit: () => void;
    google?: {
      translate: {
        TranslateElement: new (
          options: { pageLanguage: string; autoDisplay?: boolean },
          elementId: string,
        ) => void;
      };
    };
  }
}

const languages = [
  { code: "en", flag: "/assets/flags/usa.png" },
  { code: "fr", flag: "/assets/flags/france.png" },
  { code: "de", flag: "/assets/flags/Deutsch.png" },
  { code: "es", flag: "/assets/flags/Español.png" },
  { code: "pt", flag: "/assets/flags/Português.png" },
  { code: "it", flag: "/assets/flags/Italiano.png" },
  { code: "pl", flag: "/assets/flags/Polski.png" },
  { code: "zh-CN", flag: "/assets/flags/cn.png" },
  { code: "ja", flag: "/assets/flags/japan.png" },
  { code: "ar", flag: "/assets/flags/saudi-arabia.png" },
];

function getLangFromPath(pathname: string): string {
  const segment = pathname.split("/")[1];
  const match = languages.find((l) => l.code === segment);
  return match ? match.code : "en";
}

function getGoogTransCookie(): string | null {
  const match = document.cookie
    .split("; ")
    .find((row) => row.startsWith("googtrans="));
  return match ? match.split("=")[1] : null;
}

function clearCookies() {
  const expiry = "expires=Thu, 01 Jan 1970 00:00:00 UTC";
  const h = window.location.hostname;
  document.cookie = `googtrans=; ${expiry}; path=/`;
  document.cookie = `googtrans=; ${expiry}; path=/; domain=${h}`;
  document.cookie = `googtrans=; ${expiry}; path=/; domain=.${h}`;
}

function setCookieAndTranslate(lang: string) {
  clearCookies();
  if (lang !== "en") {
    const h = window.location.hostname;
    document.cookie = `googtrans=/en/${lang}; path=/`;
    document.cookie = `googtrans=/en/${lang}; path=/; domain=${h}`;
  }
  // Trigger the widget
  const interval = setInterval(() => {
    const select = document.querySelector(
      ".goog-te-combo",
    ) as HTMLSelectElement | null;
    if (select) {
      select.value = lang;
      select.dispatchEvent(new Event("change"));
      clearInterval(interval);
    }
  }, 100);
  setTimeout(() => clearInterval(interval), 5000);
}

export default function LanguageDropdown() {
  const [open, setOpen] = useState(false);
  const pathname = usePathname();

  // Derived — never stored in state
  const urlLang = getLangFromPath(pathname);
  const currentFlag =
    languages.find((l) => l.code === urlLang)?.flag || "/assets/flags/usa.png";

  // Sync Google Translate with the URL language
useEffect(() => {
  if (typeof window === "undefined") return;

  const cookie = getGoogTransCookie();

  // ✅ English page
  if (urlLang === "en") {
    if (cookie && cookie !== "/en/en") {
      clearCookies();

      // reload only once
      if (!sessionStorage.getItem("langReload")) {
        sessionStorage.setItem("langReload", "1");
        window.location.reload();
      }
    } else {
      sessionStorage.removeItem("langReload");
    }
  }

  // ✅ Other language
  else {
    setCookieAndTranslate(urlLang);
    sessionStorage.removeItem("langReload");
  }
}, [urlLang]);

  // Init Google Translate widget once
  useEffect(() => {
    window.googleTranslateElementInit = function () {
      if (window.google?.translate?.TranslateElement) {
        new window.google.translate.TranslateElement(
          { pageLanguage: "en", autoDisplay: false },
          "google_translate_element",
        );
      }
    };
  }, []);

  // Manual language selection — just writes cookies and triggers GT directly
  // No setState involved at all
  function handleSelectLanguage(lang: string) {
    setOpen(false);
    setCookieAndTranslate(lang);
  }

  return (
    <>
      <div className="relative">
        <button
          onClick={() => setOpen(!open)}
          className="overflow-hidden border border-white/20"
        >
          <Image
            src={currentFlag}
            alt={urlLang}
            width={30}
            height={40}
            className="object-cover"
          />
        </button>

        {open && (
          <div className="absolute right-0 mt-2 bg-[#00000050] rounded-2xl shadow-xl p-2 space-y-0.5 w-14">
            {languages
              .filter((lang) => lang.code !== urlLang)
              .map((lang) => (
                <button
                  key={lang.code}
                  onClick={() => handleSelectLanguage(lang.code)}
                  className="w-10 h-10 overflow-hidden hover:scale-110 transition"
                >
                  <Image
                    src={lang.flag}
                    alt={lang.code}
                    width={30}
                    height={40}
                    className="object-cover mx-auto"
                  />
                </button>
              ))}
          </div>
        )}
      </div>

      <div id="google_translate_element" className="hidden" />

      <Script
        src="//translate.google.com/translate_a/element.js?cb=googleTranslateElementInit"
        strategy="afterInteractive"
      />
    </>
  );
}

// "use client";

// import { useEffect, useState } from "react";
// import Script from "next/script";
// import Image from "next/image";
// import { usePathname } from "next/navigation";

// declare global {
//   interface Window {
//     googleTranslateElementInit: () => void;
//     google?: {
//       translate: {
//         TranslateElement: new (
//           options: { pageLanguage: string; autoDisplay?: boolean },
//           elementId: string,
//         ) => void;
//       };
//     };
//   }
// }

// const languages = [
//   { code: "en", flag: "/assets/flags/usa.png" },
//   { code: "fr", flag: "/assets/flags/france.png" },
//   { code: "de", flag: "/assets/flags/Deutsch.png" },
//   { code: "es", flag: "/assets/flags/Español.png" },
//   { code: "pt", flag: "/assets/flags/Português.png" },
//   { code: "it", flag: "/assets/flags/Italiano.png" },
//   { code: "pl", flag: "/assets/flags/Polski.png" },
//   { code: "zh-CN", flag: "/assets/flags/cn.png" },
//   { code: "ja", flag: "/assets/flags/japan.png" },
//   { code: "ar", flag: "/assets/flags/saudi-arabia.png" },
// ];

// function getLangFromPath(pathname: string): string {
//   const segment = pathname.split("/")[1];
//   const match = languages.find((l) => l.code === segment);
//   return match ? match.code : "en";
// }

// function triggerGoogleTranslate(lang: string) {
//   const interval = setInterval(() => {
//     const select = document.querySelector(
//       ".goog-te-combo",
//     ) as HTMLSelectElement | null;
//     if (select) {
//       select.value = lang;
//       select.dispatchEvent(new Event("change"));
//       clearInterval(interval);
//     }
//   }, 100);
//   setTimeout(() => clearInterval(interval), 5000);
// }

// export default function LanguageDropdown() {
//   const [open, setOpen] = useState(false);
//   const pathname = usePathname();

//   // ── Derived directly — no useState needed ──
//   const currentLang = getLangFromPath(pathname);
//   const currentFlag =
//     languages.find((l) => l.code === currentLang)?.flag ||
//     "/assets/flags/usa.png";

//   // Trigger Google Translate when URL language changes (external side effect only)
//   useEffect(() => {
//     if (currentLang !== "en") {
//       triggerGoogleTranslate(currentLang);
//     }
//   }, [currentLang]);

//   // Init Google Translate widget
//   useEffect(() => {
//     window.googleTranslateElementInit = function () {
//       if (window.google?.translate?.TranslateElement) {
//         new window.google.translate.TranslateElement(
//           { pageLanguage: "en", autoDisplay: false },
//           "google_translate_element",
//         );
//       }
//     };
//   }, []);

//   const setLanguage = (lang: string) => {
//     triggerGoogleTranslate(lang);
//     setOpen(false);
//   };

//   return (
//     <>
//       <div className="relative">
//         {/* Main Flag Button */}
//         <button
//           onClick={() => setOpen(!open)}
//           className="overflow-hidden border border-white/20"
//         >
//           <Image
//             src={currentFlag}
//             alt={currentLang}
//             width={30}
//             height={40}
//             className="object-cover"
//           />
//         </button>

//         {/* Dropdown */}
//         {open && (
//           <div className="absolute right-0 mt-2 bg-[#00000050] rounded-2xl shadow-xl p-2 space-y-0.5 w-14">
//             {languages
//               .filter((lang) => lang.code !== currentLang)
//               .map((lang) => (
//                 <button
//                   key={lang.code}
//                   onClick={() => setLanguage(lang.code)}
//                   className="w-10 h-10 overflow-hidden hover:scale-110 transition"
//                 >
//                   <Image
//                     src={lang.flag}
//                     alt={lang.code}
//                     width={30}
//                     height={40}
//                     className="object-cover mx-auto"
//                   />
//                 </button>
//               ))}
//           </div>
//         )}
//       </div>

//       {/* Hidden Google Translate */}
//       <div id="google_translate_element" className="hidden" />

//       <Script
//         src="//translate.google.com/translate_a/element.js?cb=googleTranslateElementInit"
//         strategy="afterInteractive"
//       />
//     </>
//   );
// }
