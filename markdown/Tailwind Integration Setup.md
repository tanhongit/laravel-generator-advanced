#Tailwind Integration

Taiilwind CSS is a utility-first CSS framework. 

Unlike other framework works, Tailwind CSS doesn't have ready-to-use components like bootstrap( .btn, .card, ...) but we use utility-based class to style the web page.

##Setup Tailwind CSS

Before proceeding with the installation, run `npm install` to install Laravel's front-end packages.

To install Tailwind CSS run the command.

	npm install -D tailwindcss@latest postcss@latest autoprefixer@latest

##Create configuration file

We will create a file `tailwind.config.js` at the root directory of the project.

Run the command.

	npx tailwindcss init

The `tailwind.config.js` file will have the following structure:

	/** @type {import('tailwindcss').Config} */

	module.exports = {
  
	content: [

    	"./resources/**/*.blade.php",
    	"./resources/**/*.js",
    	"./resources/**/*.vue",
 	 ],
  
	theme: {
    	extend: {},
 	},

  	plugins: [    

    require("tailwindcss"),

    ],
	}

*this config file can be custom*

##Tailwind Configuration in Laravel Mix

In the file `webpack.mix.js` we add the following:

	mix.postCss('resources/tailwind/tailwind.css', 'public/css', [ // ]);

##Add Tailwind to the CSS file

In the file `.resources/tailwind/tailwind.css` add the following lines:

	@tailwind base;
	@tailwind components;
	@tailwind utilities;

Add path to file `resources/views/layouts/app.blade.php.`


		<link href="{{ asset('css/app.css') }}" rel="stylesheet">

After you have added the file, run the command` npm run dev` or `npm run watch` (to watch for changes in the css file in the resources folder).
