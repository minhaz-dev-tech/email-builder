
<template>
  <EmailEditor v-model="content" />
</template>
  
  <script setup>
import EmailEditor from "./EmailEditor.vue";
import { ref } from "vue";

const content =
  ref(`<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr>
      <td align="center">
        <table width="600" cellpadding="0" cellspacing="0" style="background-color:#ffffff; padding:20px; border-radius:8px; margin-top:30px;">
          <!-- Header -->
          <tr>
            <td align="center" style="padding-bottom: 20px;">
              <img src="https://yourlogo.url/logo.png" alt="Logo" width="120" />
            </td>
          </tr>

          <!-- Title -->
          <tr>
            <td align="center" style="color:#333333; font-size: 24px; font-weight: bold;">
              🚀 New Opportunities Are Waiting for You!
            </td>
          </tr>

          <!-- Body -->
          <tr>
            <td style="color:#555555; font-size: 16px; line-height: 1.6; padding: 20px 0;">
              Hi <strong>[First Name]</strong>,
              <br><br>
              We’ve found exciting new jobs that match your profile. Take the next step in your career today and explore handpicked opportunities just for you.
              <br><br>
              🔹 Web Developer – Tokyo<br>
              🔹 UI/UX Designer – Remote<br>
              🔹 Project Manager – Osaka
              <br><br>
              Update your resume and apply with one click!
            </td>
          </tr>

          <!-- CTA Button -->
          <tr>
            <td align="center">
              <a href="https://yourjobportal.com/match" 
                 style="background-color:#007BFF; color:#ffffff; text-decoration:none; padding:12px 24px; border-radius:6px; font-size:16px;">
                View Jobs Now
              </a>
            </td>
          </tr>

          <!-- Footer -->
          <tr>
            <td align="center" style="font-size: 12px; color:#888888; padding-top: 30px;">
              You are receiving this email because you subscribed to job updates on <strong>YourJobPortal</strong>.
              <br>
              <a href="https://yourjobportal.com/unsubscribe" style="color:#888888;">Unsubscribe</a> |
              <a href="https://yourjobportal.com/profile" style="color:#888888;">Manage Preferences</a>
              <br><br>
              © 2025 YourJobPortal, Inc.
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>
`);
</script>
  