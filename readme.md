# JSON TeX Resume

This project abstracts LaTeX resume creation into JSON configuration definitions. 

## Template

The resume template is defined through generic `.tex` components in the [components folder](./src/components/). These are used as the building blocks for the candidate's final resume. The components are meant to use minimal static values and rely on LaTeX variable definitions to populate the resume sections. These variable values are defined in our JSON configs. 

## JSON Configuration

Most LaTeX components are not self sufficient and will not compile without their variable definitions. The definitions are housed in JSON files in the [config folder](./config/). The field names follow the required variable uses in the LaTeX template, and the values defined are reflected in the final PDF. 

The resume sections are defined in the [sections.json](./config/sections.json) file and needs to follow this format:

```json
[
    {
        "sectionName": "Experience",
        "subheadings": [
            {
                "company": "No Name Legacy Software Inc.",
                "period": "May 2023 - Next Layoff Round",
                "position": "Internal Tools Developer",
                "location": "localhost",
                "techstack": "C, COBOL",
                "rawText": "You can really just type anything you want here! how crazy is that???",
                "resumeItems": [
                    "Did this and that",
                    "Achieved this and that"
                ]
            },
            ...
        ]
    },
    ...
]
```

> All fields are optional allowing for section structure variability. E.g. Refer to `Professional Summary` section in [samples/john-doe.pdf](samples/john-doe.pdf).

[heading.json](./config/heading.json) is kept separate to define the candidate's information separate from the rest of the content. Following format is required:

```json
{
    "candidateFirstName": "Raging",
    "candidateLastName": "Chipmunk",
    "candidateAddress": "Non Doxable Address",
    "candidatePhone": "+1 (123) 456-7812",
    "candidateEmail": "email@personal.com",
    "linkedin": "linkedin",
    "github": "github",
    "portfolio": "portfolio.me"
}
```

> These fields are also optional, except for a first or last name.

Contact details are rendered as plain text (no icons) so ATS parsers can read them: a line with the address, phone and email, followed by a line of links: `linkedin.com/in/<linkedin>`, `github.com/<github>` and `<portfolio>`. Empty fields are left out.

### Embedding LaTeX

You might want to change the formatting of some text on your resume like bolding important figures or changing color of some parts of the content. 

For such purposes, you can wrap the text in LaTeX embeddings within your JSON configs. e.g. ```json "company": "\\textcolor{red}{Spotify}"```.

To fascilitate the above, the following characters aren't escaped in the script `\{}`. If you need to use them in your resume you can do the following:

- `\`: ```json\\textbackslash{}```
- `{`: ```json\\{```
- `}`: ```json\\}```


## Python Script

[generate_resume.py](./src/generate_resume.py) is the main script that glues all the components and their respective configurations together. It iterates over the configuration files, translates them into variable definitions in LaTeX, and adds all the components to one single file called `resume.tex`. 

## Create Your Resume

### Pre-requisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### Steps

1. Populate [JSON configs as defined above](#json-configuration).
2. Run `docker compose up`.
3. Find `resume.pdf` in a newly created `out` folder.

## CI/CD Pipeline

This repo comes with a configurable pipeline using repository actions' secrets and variables. This pipeline aims to build and push your resume to a different repository. This different repository can be your portfolio website or just a place you keep your most up-to-date version of your resume. 

In my personal fork, I have the pipeline pushing the resume to my [portfolio website repo](https://github.com/mbahgatTech/Portfolio-Website) which itself has a pipeline that deploys my website to Vercel. Now, if I change something in my configuration and push it to master branch, my website will have the latest resume.

### Setting Up the Pipeline

1. **Fork the Repository**: Start by forking this repository to your own GitHub account.

2. **Configure Secrets**:
   - Go to the "Settings" tab of your forked repository.
   - Navigate to "Secrets and variables" > "Actions".
   - Add the following secrets:
     - `TARGET_REPO_PAT`: A personal access token with write access to the target repository.

3. **Configure Variables**:
   - Go to the "Settings" tab of your forked repository.
   - Navigate to "Secrets and variables" > "Actions".
   - Add the following variables:
     - `GIITHUB_OWNER_REPO`: The owner and repository name of the target repository (e.g., `username/repo`).
     - `REPO_RESUME_PATH`: The path in the target repository where the resume will be stored (e.g., `public/resume.pdf`).

4. **Run the Pipeline**:
   - The pipeline is triggered automatically on every push to the `master` branch.

### Pipeline Workflow

The pipeline consists of the following steps:

1. **Checkout Repository**: Checks out the current repository.
2. **Build Docker Image**: Builds a Docker image for generating the resume.
3. **Run Docker Container**: Runs the Docker container to generate the resume and outputs it to the `out` directory.
4. **Deploy Resume to Target Repo**: Uses a custom GitHub action to clone the target repository, copy the generated resume, and push the changes to the target repository.

By following these steps, you can automate the process of generating and deploying your resume to a specified repository, ensuring that your resume is always up-to-date and easily accessible.

## License

MIT License.
