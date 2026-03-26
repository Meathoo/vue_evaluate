<template>
    <footer class="footer">
        <div class="container">
            <div class="row align-items-center justify-content-center">
                <div class="col-md-12 text-center">
                    <div class="copyright py-3">
                        &copy; {{ year }} 試卷分析系統
                    </div>
                </div>
            </div>
        </div>
    </footer>
</template>
<script>
export default {
  name: 'app-footer',
  data() {
    return {
      year: new Date().getFullYear()
    }
  }
};
</script>
<style>
</style>
