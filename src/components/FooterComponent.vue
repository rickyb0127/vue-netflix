<template>
  <div class="footer">
    <div class="social-links">
      <i class="fa-brands fa-facebook-f social-link"></i>
      <i class="fa-brands fa-instagram social-link"></i>
      <i class="fa-brands fa-twitter social-link"></i>
      <i class="fa-brands fa-youtube social-link"></i>
    </div>
    <div class="footer-links row">
      <div class="col-3">Audio Description</div>
      <div class="col-3">Help Center</div>
      <div class="col-3">Gift Cards</div>
      <div class="col-3">Media Center</div>
    </div>
    <div class="footer-links row">
      <div class="col-3">Investor Relations</div>
      <div class="col-3">Jobs</div>
      <div class="col-3">Netflix Shop</div>
      <div class="col-3">Terms of Use</div>
    </div>
    <div class="footer-links row">
      <div class="col-3">Privacy</div>
      <div class="col-3">Legal Notices</div>
      <div class="col-3">Cookie Preferences</div>
      <div class="col-3">Corporate Information</div>
    </div>
    <div class="footer-links row">
      <div class="col-3">Contact Us</div>
      <div class="col-3">Do Not Sell or Share My Personal Information</div>
    </div>
    <div>
      <span class="service-code">Service Code</span>
    </div>
  </div>
</template>

<script>

export default {
  name: 'FooterComponent'
}
</script>

<style scoped>
.footer {
  height: 300px;
  padding-left: 200px;
  padding-right: 200px;
}

.social-links {
  color: white;
  cursor: pointer;
}

.social-link {
  padding: 15px;
  font-size: 22px;
}

.footer-links {
  color: gray;
  padding-bottom: 10px;
  cursor: pointer;
}

.service-code {
  color: gray;
  border: 1px solid gray;
  padding: 5px;
  cursor: pointer;
}
</style>