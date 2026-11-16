---
description: Default instructions for the Karma plugin. Use this skill whenever this
  plugin is invoked.
name: instructions
---

Given any Resume PDF as input, generate a high-resolution, two-column Infographic Resume as a fully standalone HTML file ready for seamless PDF export, following strict professional design guidelines.
1. Layout Requirements:
• Implement a flex-based two-column structure:
•	Left Column (flex: 2): Core Content (Summary + Full Professional Timeline)
•	Right Column (flex: 1): Supplementary Content (Skills, Technologies, Productivity Focus Chart, Education)
• Responsive fallback: If screen shrinks, allow wrapping but preserve margins.
• Max-width: 1200px centered.
• Padding: 40px inside container, 30px outside margins.
2. Design Specifications:
• Background: Light neutral grey (#f4f6f8)
• Container Background: White (#ffffff), soft-rounded corners (10px radius), box-shadow (0 5px 15px rgba(0,0,0,0.1))
• Highlight Color: Golden (#DAA520)
• Typography: Roboto (linked via Google Fonts), fallback to Helvetica Neue and Open Sans
• Font Sizes:
•	Header: 32px
•	Section Header: 24px with golden underline
•	Body Text: 16px minimum
• Elements:
•	Timeline Events: Dot markers with heading-date pairs
•	Badges (Skills, Technologies): Rounded badges, golden background, flex-wrap enabled
•	Chart: Static, aesthetic Pie Chart via Chart.js (dynamic values — not hardcoded)
• No forced page breaks — allow natural scroll.
3. Content Structuring:
a) Header Section:
• Name (h1) + 1-line Professional Tagline
• Contact Information (Email, Phone, LinkedIn, Website, Blog)
b) Summary Section:
• Extract 4–6 lines describing the candidate profile.
• If absent, infer sensibly from career history.
c) Professional Experience Section:
• Full list of roles, latest first.
• Each experience includes:
•	Title
•	Organization
•	Period (Start–End)
•	3–5 action-result-oriented bullet points.
• Group early-career roles under 'Earlier Roles' (for bulk years like 1999–2011).
d) Skills Section:
• Extract all technical and soft skills.
• Display each skill as a rounded golden badge.
e) Technologies Section:
• List programming languages, frameworks, cloud platforms, tools, etc.
• Display each technology as a rounded golden badge under “Technologies”.
f) Productivity Focus Chart:
•	Mandatory inclusion of a static Pie Chart via Chart.js.
•	No hardcoded sample values.
•	Values and labels must be logically guessed from the candidate's profile following clear reasoning.
Productivity Focus Guessing Strategy:
•	Analyze resume experience and roles.
•	Guess major professional focus areas such as:
o	Research & Innovation
o	Leadership & Strategy
o	Data Science & Analytics
o	Teaching & Writing
o	Client Engagement
o	Strategic Planning
o	Personal Development
•	Allocate approximate percentage splits totaling 100% based on role emphasis.
•	Labels and values must feel natural, professional, and aligned with candidate’s work history.
Chart Properties (Fixed Aesthetics):
•	Chart Size: 250px width × 250px height (explicitly set in the <canvas> tag)
•	Background Color Palette: Soft golden and orange variants
o	e.g., #DAA520, #f39c12, #e67e22, #d35400, #c0392b
•	Border: White (#ffffff) border for each pie slice
•	Hover Effect: Hover offset (20px) for lively popout
•	Legend: Displayed below the chart
•	Responsive Setting: false (fixed size maintained for clean PDF export)
Standard Chart Embedding Format:
<canvas id="productivityChart" width="250" height="250"></canvas>
<script>
const ctx = document.getElementById('productivityChart').getContext('2d');
const productivityChart = new Chart(ctx, {
  type: 'pie',
  data: {
    labels: [/* dynamically inferred labels */],
    datasets: [{
      label: 'Productivity Split',
      data: [/* dynamically inferred percentages totaling 100 */],
      backgroundColor: ['#DAA520', '#f39c12', '#e67e22', '#d35400', '#c0392b'],
      borderColor: ['#ffffff'],
      borderWidth: 2,
      hoverOffset: 20
    }]
  },
  options: {
    responsive: false,
    plugins: {
      legend: { position: 'bottom' },
      tooltip: { enabled: true }
    }
  }
});
</script>
g) Education Section:
• List all educational qualifications.
• Format each: Degree Name, Institution, Period.
4. Export Standards:
• Produce standalone HTML including:
•	Linked Roboto font (via Google Fonts)
•	Linked Chart.js library via CDN
• No external JS libraries other than Chart.js.
• Ready for direct browser print/save to high-resolution PDF.
• Fonts embedded to ensure print compatibility.
5. Critical Quality Checklist Before Output:
✅ Flex-based two-column responsive structure
✅ Golden highlights (#DAA520) for important elements
✅ Soft clean shadows on container
✅ Full professional experience including early roles (bulk grouping if needed)
✅ Badges for Skills and Technologies
✅ Mandatory Productivity Focus Pie Chart with dynamically guessed data
✅ Fixed size, hover effects, legend shown for the Chart
✅ Smooth scrolling and natural A4/PDF export
✅ No blank sections even if some data absent
✅ Margins safe for professional printing
📌 Output Requirement:
• Deliver full standalone HTML code only —
• Must be immediately usable without modification for seamless PDF export.
✨ Bonus Instruction:
•	Even if the resume does not provide explicit productivity focus data, always intelligently guess and include the Chart.
•	Never omit the Pie Chart.
•	Ensure that the Pie Chart feels professionally reflective of the candidate’s profile.