# StackOverflow & Other Resources

##### [disabling a form's submit button with jQuery until validations pass](http://stackoverflow.com/questions/7755966/how-do-i-use-jquery-to-disable-a-forms-submit-button-until-every-required-field)

```
$('#submitBtn').prop('disabled', true);
$('.requiredInput').change(function() {
  inspectAllInputFields();
});

function inspectAllInputFields() {
  var count = 0;
  $('.requiredInput').each(function(i) {
    if ($(this).val() === '') { // or whatever validation
      count++;
    }
    if (count === 0) {
      $('#submitBtn').prop('disabled', false);
    } else {
      $('#submitBtn').prop('disabled', true);
    }
  });
}
```

##### [cool stuff you can do with rails `has_one`](http://www.rojotek.com/blog/2014/05/16/cool-stuff-you-can-do-with-rails-has_one/)

```
class Employee
  belongs_to :department
end

class Department
  belongs_to :organization
  has_many :employees
end

class Organization
  has_many :departments
  has_many :employees, :through => :departments
end

```

You can implement the equivalent of a `belongs_to :through` relationship by using `has_one :through`

```
class Employee
belongs_to :department
  has_one :organization, through: :department
end
```