<template>
  <footer class="footer">
    <div class="container-fluid">
      <ul class="nav">
        <li class="nav-item">
          <a class="nav-link">
            <i>NOTICE: Use contextually-aware output encoding. Only <b>you</b> can prevent payload fires.</i>
          </a>
        </li>
      </ul>
      <div class="copyright">
         Forked from the original author:<a href="https://x.com/IAmMandatory" target="_blank"><code>@IAmMandatory</code></a>
      </div>
    </div>
  </footer>
</template>
<script>
  export default {
    data() {
      return {
        year: new Date().getFullYear()
      }
    }
  };
</script>
<style>
</style>
