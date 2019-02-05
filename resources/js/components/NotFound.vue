<template>
    <div class="container">
        <section class="content d-flex align-items-center justify-content-center">
            <div class="error-page">
                <h2 class="headline text-danger"> 404</h2>

                <div class="error-content">
                <h3><i class="fa fa-exclamation-triangle text-danger"></i> Oops! Page not found.</h3>

                <p>
                    We could not find the page you were looking for.
                    Meanwhile, you may <router-link to="/dashboard">return to dashboard</router-link> or try using the search form.
                </p>
                </div>
                <!-- /.error-content -->
            </div>
            <!-- /.error-page -->
        </section>
    </div>
</template>

<script>
    export default {
        mounted() {
            console.log('Component mounted.')
        }
    }
</script>
