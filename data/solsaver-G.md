## Invert if condition to save gas

```
    if (
      state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
        || state == ActionState.Failed
    ) revert CannotCancelInState(state);
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L264-L267

Fix:
```
-    if (
-      state == ActionState.Executed || state == ActionState.Canceled || state == ActionState.Expired
-        || state == ActionState.Failed
-    ) revert CannotCancelInState(state);
+    if (state != ActionState.Active && state != ActionState.Approved && state != ActionState.Queued) revert CannotCancelInState(state);
```

## Compress JSON to save gas

The JSON can be compressed to save characters, thereby reducing the gas.

```
    parts[0] = '{ "name": "Llama Policies: ';
    parts[1] = name;
    parts[2] = '", "description": "This collection includes all members of the Llama organization: ';
    parts[3] = name;
    parts[4] =
      '. Visit https://app.llama.xyz to learn more.", "image":"https://llama.xyz/policy-nft/llama-profile.png", "external_link": "https://app.llama.xyz", "banner":"https://llama.xyz/policy-nft/llama-banner.png" }';
```
Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L106-L111

Fix:

```
    parts[0] = '{"name":"Llama Policies: ';
    parts[1] = name;
    parts[2] = '","description":"This collection includes all members of the Llama organization: ';
    parts[3] = name;
    parts[4] =
      '. Visit https://app.llama.xyz to learn more.","image":"https://llama.xyz/policy-nft/llama-profile.png","external_link":"https://app.llama.xyz","banner":"https://llama.xyz/policy-nft/llama-banner.png"}';
```

Other instance:

```
          '{"name": "',
          name,
          ' Member", "description": "This NFT represents membership in the Llama organization: ',
          name,
          ". The owner of this NFT can participate in governance according to their roles and permissions. Visit https://app.llama.xyz/profiles/",
          policyholder,
          ' to view their profile page.", "external_url": "https://app.llama.xyz", "image": "data:image/svg+xml;base64,',
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L85-L91

## Use string.concat directly without creating parts array

There is no need to create a parts array to concat the string below.

```
    string[5] memory parts;
    parts[0] = '{ "name": "Llama Policies: ';
    parts[1] = name;
    parts[2] = '", "description": "This collection includes all members of the Llama organization: ';
    parts[3] = name;
    parts[4] =
      '. Visit https://app.llama.xyz to learn more.", "image":"https://llama.xyz/policy-nft/llama-profile.png", "external_link": "https://app.llama.xyz", "banner":"https://llama.xyz/policy-nft/llama-banner.png" }';
    string memory json = Base64.encode(bytes(string.concat(parts[0], parts[1], parts[2], parts[3], parts[4])));
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L105-L112

Fix:

```
        string memory json = Base64.encode(bytes(string.concat(
            '{ "name": "Llama Policies: ',
            name,
            '", "description": "This collection includes all members of the Llama organization: ',
            name,
            '. Visit https://app.llama.xyz to learn more.", "image":"https://llama.xyz/policy-nft/llama-profile.png", "external_link": "https://app.llama.xyz", "banner":"https://llama.xyz/policy-nft/llama-banner.png" }'
        )));
        return string.concat("data:application/json;base64,", json);
```

Another instance:

```
    parts[0] =
      '<svg xmlns="http://www.w3.org/2000/svg" width="390" height="500" fill="none"><g clip-path="url(#a)"><rect width="390" height="500" fill="#0B101A" rx="13.393" /><mask id="b" width="364" height="305" x="4" y="30" maskUnits="userSpaceOnUse" style="mask-type:alpha"><ellipse cx="186.475" cy="182.744" fill="#8000FF" rx="196.994" ry="131.329" transform="rotate(-31.49 186.475 182.744)" /></mask><g mask="url(#b)"><g filter="url(#c)"><ellipse cx="226.274" cy="247.516" fill="url(#d)" rx="140.048" ry="59.062" transform="rotate(-31.49 226.274 247.516)" /></g><g filter="url(#e)"><ellipse cx="231.368" cy="254.717" fill="url(#f)" rx="102.858" ry="43.378" transform="rotate(-31.49 231.368 254.717)" /></g></g><g filter="url(#g)"><ellipse cx="237.625" cy="248.969" fill="url(#h)" rx="140.048" ry="59.062" transform="rotate(-31.49 237.625 248.969)" /></g><circle cx="109.839" cy="147.893" r="22" fill="url(#i)" /><rect width="150" height="35.071" x="32" y="376.875" fill="';


    parts[1] = color;


    parts[2] =
      '" rx="17.536" /><text xml:space="preserve" fill="#0B101A" font-family="ui-monospace,Cascadia Mono,Menlo,Monaco,Segoe UI Mono,Roboto Mono,Oxygen Mono,Ubuntu Monospace,Source Code Pro,Droid Sans Mono,Fira Mono,Courier,monospace" font-size="16"><tspan x="45.393" y="399.851">';


    parts[3] = string.concat(LibString.slice(policyholder, 0, 6), "...", LibString.slice(policyholder, 38, 42));


    parts[4] =
      '</tspan></text><path fill="#fff" d="M341 127.067a11.433 11.433 0 0 0 8.066-8.067 11.436 11.436 0 0 0 8.067 8.067 11.433 11.433 0 0 0-8.067 8.066 11.43 11.43 0 0 0-8.066-8.066Z" /><path stroke="#fff" stroke-width="1.5" d="M349.036 248.018V140.875" /><circle cx="349.036" cy="259.178" r="4.018" fill="#fff" /><path stroke="#fff" stroke-width="1.5" d="M349.036 292.214v-21.429" /><path fill="#fff" d="M343.364 33.506a1.364 1.364 0 0 0-2.728 0V43.85l-7.314-7.314a1.364 1.364 0 0 0-1.929 1.928l7.315 7.315h-10.344a1.364 1.364 0 0 0 0 2.727h10.344l-7.315 7.315a1.365 1.365 0 0 0 1.929 1.928l7.314-7.314v10.344a1.364 1.364 0 0 0 2.728 0V50.435l7.314 7.314a1.364 1.364 0 0 0 1.929-1.928l-7.315-7.315h10.344a1.364 1.364 0 1 0 0-2.727h-10.344l7.315-7.315a1.365 1.365 0 0 0-1.929-1.928l-7.314 7.314V33.506ZM73.81 44.512h-4.616v1.932h1.777v10.045h-2.29v1.932h6.82V56.49h-1.69V44.512ZM82.469 44.512h-4.617v1.932h1.777v10.045h-2.29v1.932h6.82V56.49h-1.69V44.512ZM88.847 51.534c.097-.995.783-1.526 2.02-1.526 1.236 0 1.854.531 1.854 1.68v.28l-3.4.416c-2.02.251-3.603 1.13-3.603 3.11 0 1.971 1.497 3.101 3.767 3.101 1.903 0 2.743-.724 3.14-1.343h.192v1.17h2.647v-6.337c0-2.763-1.777-4.009-4.54-4.009-2.782 0-4.482 1.246-4.685 3.168v.29h2.608Zm-.338 3.835c0-.763.58-1.13 1.42-1.246l2.792-.367v.435c0 1.632-1.082 2.453-2.57 2.453-1.043 0-1.642-.502-1.642-1.275ZM97.614 58.42h2.608v-6.51c0-1.246.657-1.787 1.575-1.787.821 0 1.226.474 1.226 1.275v7.023h2.609v-6.51c0-1.247.656-1.788 1.564-1.788.831 0 1.227.474 1.227 1.275v7.023h2.618v-7.38c0-1.835-1.159-2.927-2.927-2.927-1.584 0-2.318.686-2.743 1.44h-.194c-.289-.657-1.004-1.44-2.472-1.44-1.44 0-2.067.6-2.415 1.208h-.193v-1.015h-2.483v10.114ZM115.654 51.534c.097-.995.782-1.526 2.019-1.526 1.236 0 1.854.531 1.854 1.68v.28l-3.4.416c-2.019.251-3.603 1.13-3.603 3.11 0 1.971 1.498 3.101 3.767 3.101 1.903 0 2.744-.724 3.14-1.343h.193v1.17h2.647v-6.337c0-2.763-1.778-4.009-4.54-4.009-2.782 0-4.482 1.246-4.685 3.168v.29h2.608Zm-.338 3.835c0-.763.58-1.13 1.42-1.246l2.791-.367v.435c0 1.632-1.081 2.453-2.569 2.453-1.043 0-1.642-.502-1.642-1.275ZM35.314 52.07a.906.906 0 0 1 .88-.895h11.72a4.205 4.205 0 0 0 3.896-2.597 4.22 4.22 0 0 0 .323-1.614V32h-3.316v14.964a.907.907 0 0 1-.88.894H36.205a4.206 4.206 0 0 0-2.972 1.235A4.219 4.219 0 0 0 32 52.07v10.329h3.314v-10.33ZM53.6 34.852h-.147l.141.14v3.086h3.05l1.43 1.446a4.21 4.21 0 0 0-2.418 1.463 4.222 4.222 0 0 0-.95 2.664v18.752h3.3V43.647a.909.909 0 0 1 .894-.895h.508c1.947 0 2.608-1.086 2.803-1.543.196-.456.498-1.7-.88-3.085l-3.23-3.261h-1.006" /><path fill="#fff" d="M44.834 60.77a5.448 5.448 0 0 1 3.89 1.629h4.012a8.8 8.8 0 0 0-3.243-3.608 8.781 8.781 0 0 0-12.562 3.608h4.012a5.459 5.459 0 0 1 3.89-1.629Z" />';


    parts[5] = logo;


    parts[6] =
      '</g><defs><radialGradient id="d" cx="0" cy="0" r="1" gradientTransform="rotate(-90.831 270.037 36.188) scale(115.966 274.979)" gradientUnits="userSpaceOnUse"><stop stop-color="';


    parts[7] = color;


    parts[8] = '" /><stop offset="1" stop-color="';


    parts[9] = color;


    parts[10] =
      '" stop-opacity="0" /></radialGradient><radialGradient id="f" cx="0" cy="0" r="1" gradientTransform="matrix(7.1866 -72.99558 127.41796 12.54463 239.305 292.746)" gradientUnits="userSpaceOnUse"><stop stop-color="';


    parts[11] = color;


    parts[12] = '" /><stop offset="1" stop-color="';


    parts[13] = color;


    parts[14] =
      '" stop-opacity="0" /></radialGradient><radialGradient id="h" cx="0" cy="0" r="1" gradientTransform="rotate(-94.142 264.008 51.235) scale(212.85 177.126)" gradientUnits="userSpaceOnUse"><stop stop-color="';


    parts[15] = color;


    parts[16] = '" /><stop offset="1" stop-color="';


    parts[17] = color;


    parts[18] =
      '" stop-opacity="0" /></radialGradient><radialGradient id="i" cx="0" cy="0" r="1" gradientTransform="matrix(23.59563 32 -33.15047 24.44394 98.506 137.893)" gradientUnits="userSpaceOnUse"><stop stop-color="#0B101A" /><stop offset=".609" stop-color="';


    parts[19] = color;


    parts[20] =
      '" /><stop offset="1" stop-color="#fff" /></radialGradient><filter id="c" width="346.748" height="277.643" x="52.9" y="108.695" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="25" /></filter><filter id="e" width="221.224" height="170.469" x="120.757" y="169.482" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="10" /></filter><filter id="g" width="446.748" height="377.643" x="14.251" y="60.147" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="50" /></filter><clipPath id="a"><rect width="390" height="500" fill="#fff" rx="13.393" /></clipPath></defs></svg>';


    // This output has been broken up into multiple outputs to avoid a stack too deep error
    string memory output1 =
      string.concat(parts[0], parts[1], parts[2], parts[3], parts[4], parts[5], parts[6], parts[7], parts[8]);
    string memory output2 =
      string.concat(parts[9], parts[10], parts[11], parts[12], parts[13], parts[14], parts[15], parts[16], parts[17]);
    string memory output = string.concat(output1, output2, parts[18], parts[19], parts[20]);
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadata.sol#L25-L80

Fix:

```
        string memory output1 = string.concat(
            '<svg xmlns="http://www.w3.org/2000/svg" width="390" height="500" fill="none"><g clip-path="url(#a)"><rect width="390" height="500" fill="#0B101A" rx="13.393" /><mask id="b" width="364" height="305" x="4" y="30" maskUnits="userSpaceOnUse" style="mask-type:alpha"><ellipse cx="186.475" cy="182.744" fill="#8000FF" rx="196.994" ry="131.329" transform="rotate(-31.49 186.475 182.744)" /></mask><g mask="url(#b)"><g filter="url(#c)"><ellipse cx="226.274" cy="247.516" fill="url(#d)" rx="140.048" ry="59.062" transform="rotate(-31.49 226.274 247.516)" /></g><g filter="url(#e)"><ellipse cx="231.368" cy="254.717" fill="url(#f)" rx="102.858" ry="43.378" transform="rotate(-31.49 231.368 254.717)" /></g></g><g filter="url(#g)"><ellipse cx="237.625" cy="248.969" fill="url(#h)" rx="140.048" ry="59.062" transform="rotate(-31.49 237.625 248.969)" /></g><circle cx="109.839" cy="147.893" r="22" fill="url(#i)" /><rect width="150" height="35.071" x="32" y="376.875" fill="',
            color,
            '" rx="17.536" /><text xml:space="preserve" fill="#0B101A" font-family="ui-monospace,Cascadia Mono,Menlo,Monaco,Segoe UI Mono,Roboto Mono,Oxygen Mono,Ubuntu Monospace,Source Code Pro,Droid Sans Mono,Fira Mono,Courier,monospace" font-size="16"><tspan x="45.393" y="399.851">',
            string.concat(LibString.slice(policyholder, 0, 6), "...", LibString.slice(policyholder, 38, 42)),        
            '</tspan></text><path fill="#fff" d="M341 127.067a11.433 11.433 0 0 0 8.066-8.067 11.436 11.436 0 0 0 8.067 8.067 11.433 11.433 0 0 0-8.067 8.066 11.43 11.43 0 0 0-8.066-8.066Z" /><path stroke="#fff" stroke-width="1.5" d="M349.036 248.018V140.875" /><circle cx="349.036" cy="259.178" r="4.018" fill="#fff" /><path stroke="#fff" stroke-width="1.5" d="M349.036 292.214v-21.429" /><path fill="#fff" d="M343.364 33.506a1.364 1.364 0 0 0-2.728 0V43.85l-7.314-7.314a1.364 1.364 0 0 0-1.929 1.928l7.315 7.315h-10.344a1.364 1.364 0 0 0 0 2.727h10.344l-7.315 7.315a1.365 1.365 0 0 0 1.929 1.928l7.314-7.314v10.344a1.364 1.364 0 0 0 2.728 0V50.435l7.314 7.314a1.364 1.364 0 0 0 1.929-1.928l-7.315-7.315h10.344a1.364 1.364 0 1 0 0-2.727h-10.344l7.315-7.315a1.365 1.365 0 0 0-1.929-1.928l-7.314 7.314V33.506ZM73.81 44.512h-4.616v1.932h1.777v10.045h-2.29v1.932h6.82V56.49h-1.69V44.512ZM82.469 44.512h-4.617v1.932h1.777v10.045h-2.29v1.932h6.82V56.49h-1.69V44.512ZM88.847 51.534c.097-.995.783-1.526 2.02-1.526 1.236 0 1.854.531 1.854 1.68v.28l-3.4.416c-2.02.251-3.603 1.13-3.603 3.11 0 1.971 1.497 3.101 3.767 3.101 1.903 0 2.743-.724 3.14-1.343h.192v1.17h2.647v-6.337c0-2.763-1.777-4.009-4.54-4.009-2.782 0-4.482 1.246-4.685 3.168v.29h2.608Zm-.338 3.835c0-.763.58-1.13 1.42-1.246l2.792-.367v.435c0 1.632-1.082 2.453-2.57 2.453-1.043 0-1.642-.502-1.642-1.275ZM97.614 58.42h2.608v-6.51c0-1.246.657-1.787 1.575-1.787.821 0 1.226.474 1.226 1.275v7.023h2.609v-6.51c0-1.247.656-1.788 1.564-1.788.831 0 1.227.474 1.227 1.275v7.023h2.618v-7.38c0-1.835-1.159-2.927-2.927-2.927-1.584 0-2.318.686-2.743 1.44h-.194c-.289-.657-1.004-1.44-2.472-1.44-1.44 0-2.067.6-2.415 1.208h-.193v-1.015h-2.483v10.114ZM115.654 51.534c.097-.995.782-1.526 2.019-1.526 1.236 0 1.854.531 1.854 1.68v.28l-3.4.416c-2.019.251-3.603 1.13-3.603 3.11 0 1.971 1.498 3.101 3.767 3.101 1.903 0 2.744-.724 3.14-1.343h.193v1.17h2.647v-6.337c0-2.763-1.778-4.009-4.54-4.009-2.782 0-4.482 1.246-4.685 3.168v.29h2.608Zm-.338 3.835c0-.763.58-1.13 1.42-1.246l2.791-.367v.435c0 1.632-1.081 2.453-2.569 2.453-1.043 0-1.642-.502-1.642-1.275ZM35.314 52.07a.906.906 0 0 1 .88-.895h11.72a4.205 4.205 0 0 0 3.896-2.597 4.22 4.22 0 0 0 .323-1.614V32h-3.316v14.964a.907.907 0 0 1-.88.894H36.205a4.206 4.206 0 0 0-2.972 1.235A4.219 4.219 0 0 0 32 52.07v10.329h3.314v-10.33ZM53.6 34.852h-.147l.141.14v3.086h3.05l1.43 1.446a4.21 4.21 0 0 0-2.418 1.463 4.222 4.222 0 0 0-.95 2.664v18.752h3.3V43.647a.909.909 0 0 1 .894-.895h.508c1.947 0 2.608-1.086 2.803-1.543.196-.456.498-1.7-.88-3.085l-3.23-3.261h-1.006" /><path fill="#fff" d="M44.834 60.77a5.448 5.448 0 0 1 3.89 1.629h4.012a8.8 8.8 0 0 0-3.243-3.608 8.781 8.781 0 0 0-12.562 3.608h4.012a5.459 5.459 0 0 1 3.89-1.629Z" />',
            logo,
            '</g><defs><radialGradient id="d" cx="0" cy="0" r="1" gradientTransform="rotate(-90.831 270.037 36.188) scale(115.966 274.979)" gradientUnits="userSpaceOnUse"><stop stop-color="',
            color,
            '" /><stop offset="1" stop-color="'
        );
        string memory output2 = string.concat(
            color,
            '" stop-opacity="0" /></radialGradient><radialGradient id="f" cx="0" cy="0" r="1" gradientTransform="matrix(7.1866 -72.99558 127.41796 12.54463 239.305 292.746)" gradientUnits="userSpaceOnUse"><stop stop-color="',
            color,
            '" /><stop offset="1" stop-color="',
            color,
            '" stop-opacity="0" /></radialGradient><radialGradient id="h" cx="0" cy="0" r="1" gradientTransform="rotate(-94.142 264.008 51.235) scale(212.85 177.126)" gradientUnits="userSpaceOnUse"><stop stop-color="',
            color,
            '" /><stop offset="1" stop-color="',
            color
        );
        string memory output = string.concat(
            output1,
            output2,
            '" stop-opacity="0" /></radialGradient><radialGradient id="i" cx="0" cy="0" r="1" gradientTransform="matrix(23.59563 32 -33.15047 24.44394 98.506 137.893)" gradientUnits="userSpaceOnUse"><stop stop-color="#0B101A" /><stop offset=".609" stop-color="',
            color,
            '" /><stop offset="1" stop-color="#fff" /></radialGradient><filter id="c" width="346.748" height="277.643" x="52.9" y="108.695" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="25" /></filter><filter id="e" width="221.224" height="170.469" x="120.757" y="169.482" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="10" /></filter><filter id="g" width="446.748" height="377.643" x="14.251" y="60.147" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse"><feFlood flood-opacity="0" result="BackgroundImageFix" /><feBlend in="SourceGraphic" in2="BackgroundImageFix" result="shape" /><feGaussianBlur result="effect1_foregroundBlur_260_71" stdDeviation="50" /></filter><clipPath id="a"><rect width="390" height="500" fill="#fff" rx="13.393" /></clipPath></defs></svg>'
        );
```

## Save some gas by removing redundant strings

`rootColor` and `rootLogo` are not required to be defined, and can just be used inline in the `setColor` and `setLogo` methods to save some gas.

```
    string memory rootColor = "#6A45EC";
    string memory rootLogo =
      '<g><path fill="#fff" d="M91.749 446.038H85.15v2.785h2.54v14.483h-3.272v2.785h9.746v-2.785h-2.416v-17.268ZM104.122 446.038h-6.598v2.785h2.54v14.483h-3.271v2.785h9.745v-2.785h-2.416v-17.268ZM113.237 456.162c.138-1.435 1.118-2.2 2.885-2.2 1.767 0 2.651.765 2.651 2.423v.403l-4.859.599c-2.885.362-5.149 1.63-5.149 4.484 0 2.841 2.14 4.47 5.383 4.47 2.72 0 3.921-1.044 4.487-1.935h.276v1.685h3.782v-9.135c0-3.983-2.54-5.78-6.488-5.78-3.975 0-6.404 1.797-6.694 4.568v.418h3.726Zm-.483 5.528c0-1.1.829-1.629 2.03-1.796l3.989-.529v.626c0 2.354-1.546 3.537-3.672 3.537-1.491 0-2.347-.724-2.347-1.838ZM125.765 466.091h3.727v-9.386c0-1.796.938-2.576 2.25-2.576 1.173 0 1.753.682 1.753 1.838v10.124h3.727v-9.386c0-1.796.939-2.576 2.236-2.576 1.187 0 1.753.682 1.753 1.838v10.124h3.741v-10.639c0-2.646-1.657-4.22-4.183-4.22-2.264 0-3.312.989-3.92 2.075h-.276c-.414-.947-1.436-2.075-3.534-2.075-2.056 0-2.954.864-3.45 1.741h-.277v-1.462h-3.547v14.58ZM151.545 456.162c.138-1.435 1.118-2.2 2.885-2.2 1.767 0 2.65.765 2.65 2.423v.403l-4.859.599c-2.885.362-5.149 1.63-5.149 4.484 0 2.841 2.14 4.47 5.384 4.47 2.719 0 3.92-1.044 4.486-1.935h.276v1.685H161v-9.135c0-3.983-2.54-5.78-6.488-5.78-3.975 0-6.404 1.797-6.694 4.568v.418h3.727Zm-.484 5.528c0-1.1.829-1.629 2.03-1.796l3.989-.529v.626c0 2.354-1.546 3.537-3.672 3.537-1.491 0-2.347-.724-2.347-1.838Z"/><g fill="#6A45EC"><path d="M36.736 456.934c.004-.338.137-.661.372-.901.234-.241.552-.38.886-.389h16.748a5.961 5.961 0 0 0 2.305-.458 6.036 6.036 0 0 0 3.263-3.287c.303-.737.46-1.528.46-2.326V428h-4.738v21.573c-.004.337-.137.66-.372.901-.234.24-.552.379-.886.388H38.01a5.984 5.984 0 0 0-4.248 1.781A6.108 6.108 0 0 0 32 456.934v14.891h4.736v-14.891ZM62.868 432.111h-.21l.2.204v4.448h4.36l2.043 2.084a6.008 6.008 0 0 0-3.456 2.109 6.12 6.12 0 0 0-1.358 3.841v27.034h4.717v-27.04c.005-.341.14-.666.38-.907.237-.24.56-.378.897-.383h.726c2.783 0 3.727-1.566 4.006-2.224.28-.658.711-2.453-1.257-4.448l-4.617-4.702h-1.437M50.34 469.477a7.728 7.728 0 0 1 3.013.61c.955.403 1.82.994 2.547 1.738h5.732a12.645 12.645 0 0 0-4.634-5.201 12.467 12.467 0 0 0-6.658-1.93c-2.355 0-4.662.669-6.659 1.93a12.644 12.644 0 0 0-4.634 5.201h5.733a7.799 7.799 0 0 1 2.546-1.738 7.728 7.728 0 0 1 3.014-.61Z"/></g></g>';
    setColor(ROOT_LLAMA_EXECUTOR, rootColor);
    setLogo(ROOT_LLAMA_EXECUTOR, rootLogo);
```

Link: https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicyMetadataParamRegistry.sol#L61-L65

Fix:

```
    constructor(uint256 rootLlamaExecutor) {
        ROOT_LLAMA_EXECUTOR = rootLlamaExecutor;
        LLAMA_FACTORY = msg.sender;
        setColor(ROOT_LLAMA_EXECUTOR, "#6A45EC");
        setLogo(ROOT_LLAMA_EXECUTOR, '<g><path fill="#fff" d="M91.749 446.038H85.15v2.785h2.54v14.483h-3.272v2.785h9.746v-2.785h-2.416v-17.268ZM104.122 446.038h-6.598v2.785h2.54v14.483h-3.271v2.785h9.745v-2.785h-2.416v-17.268ZM113.237 456.162c.138-1.435 1.118-2.2 2.885-2.2 1.767 0 2.651.765 2.651 2.423v.403l-4.859.599c-2.885.362-5.149 1.63-5.149 4.484 0 2.841 2.14 4.47 5.383 4.47 2.72 0 3.921-1.044 4.487-1.935h.276v1.685h3.782v-9.135c0-3.983-2.54-5.78-6.488-5.78-3.975 0-6.404 1.797-6.694 4.568v.418h3.726Zm-.483 5.528c0-1.1.829-1.629 2.03-1.796l3.989-.529v.626c0 2.354-1.546 3.537-3.672 3.537-1.491 0-2.347-.724-2.347-1.838ZM125.765 466.091h3.727v-9.386c0-1.796.938-2.576 2.25-2.576 1.173 0 1.753.682 1.753 1.838v10.124h3.727v-9.386c0-1.796.939-2.576 2.236-2.576 1.187 0 1.753.682 1.753 1.838v10.124h3.741v-10.639c0-2.646-1.657-4.22-4.183-4.22-2.264 0-3.312.989-3.92 2.075h-.276c-.414-.947-1.436-2.075-3.534-2.075-2.056 0-2.954.864-3.45 1.741h-.277v-1.462h-3.547v14.58ZM151.545 456.162c.138-1.435 1.118-2.2 2.885-2.2 1.767 0 2.65.765 2.65 2.423v.403l-4.859.599c-2.885.362-5.149 1.63-5.149 4.484 0 2.841 2.14 4.47 5.384 4.47 2.719 0 3.92-1.044 4.486-1.935h.276v1.685H161v-9.135c0-3.983-2.54-5.78-6.488-5.78-3.975 0-6.404 1.797-6.694 4.568v.418h3.727Zm-.484 5.528c0-1.1.829-1.629 2.03-1.796l3.989-.529v.626c0 2.354-1.546 3.537-3.672 3.537-1.491 0-2.347-.724-2.347-1.838Z"/><g fill="#6A45EC"><path d="M36.736 456.934c.004-.338.137-.661.372-.901.234-.241.552-.38.886-.389h16.748a5.961 5.961 0 0 0 2.305-.458 6.036 6.036 0 0 0 3.263-3.287c.303-.737.46-1.528.46-2.326V428h-4.738v21.573c-.004.337-.137.66-.372.901-.234.24-.552.379-.886.388H38.01a5.984 5.984 0 0 0-4.248 1.781A6.108 6.108 0 0 0 32 456.934v14.891h4.736v-14.891ZM62.868 432.111h-.21l.2.204v4.448h4.36l2.043 2.084a6.008 6.008 0 0 0-3.456 2.109 6.12 6.12 0 0 0-1.358 3.841v27.034h4.717v-27.04c.005-.341.14-.666.38-.907.237-.24.56-.378.897-.383h.726c2.783 0 3.727-1.566 4.006-2.224.28-.658.711-2.453-1.257-4.448l-4.617-4.702h-1.437M50.34 469.477a7.728 7.728 0 0 1 3.013.61c.955.403 1.82.994 2.547 1.738h5.732a12.645 12.645 0 0 0-4.634-5.201 12.467 12.467 0 0 0-6.658-1.93c-2.355 0-4.662.669-6.659 1.93a12.644 12.644 0 0 0-4.634 5.201h5.733a7.799 7.799 0 0 1 2.546-1.738 7.728 7.728 0 0 1 3.014-.61Z"/></g></g>');
    }
```