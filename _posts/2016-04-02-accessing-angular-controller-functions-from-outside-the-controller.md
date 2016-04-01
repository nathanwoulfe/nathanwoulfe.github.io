---
layout: post
category: blog
title: Accessing Angular controller functions from outside the controller
---

````js
document.addEventListener('DOMContentLoaded', function (e) {
  $('a.form-link').on('click', function(e) {
    var $parent = angular.element(document.querySelectorAll('.credit-calculator')[0]).scope(),
      elm = angular.element(document.querySelectorAll('.crm-form-widget')[0]),
      $scope = elm.scope();

    var injector = elm.injector();
    var contentService = injector.get('contentService');

    $scope.formData['incident.c$custom_field_1'] = $parent.calc['External program name'];
    $scope.formData['incident.c$custom_field_2'] = $parent.calc['Institution name'];

    var q = {
        contentTypes: ['programItem'],
        count: 3000,
        query: {
            isArchived: 'false'
        }
    };
    contentService.get(q).then(function (resp) {
        $scope.uscPrograms  = resp;
    });
  });
});
</script>
````
