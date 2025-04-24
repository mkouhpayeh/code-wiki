# Localized Custom Validation

## Attribute Usage
```
public class ChangePasswordModel
{
    public string Email { get; set; }

    [FieldIsRequired(nameof(CurrentPassword))]
    public string CurrentPassword { get; set; }

    [FieldIsRequired(nameof(NewPassword))]
    [PasswordRequirements]
    public string NewPassword { get; set; }

    [CompareText(nameof(NewPassword), nameof(ConfirmPassword))]
    public string ConfirmPassword { get; set; }
}
```

``` cs title="Resource Files Model"
public class SharedValidationAttribute
{
}
```

## Compare Text Attribute
``` cs title="CompareText"
public class CompareTextAttribute : ValidationAttribute
{
    private readonly string _fieldToCompare;
    private readonly string _currentField;
    private readonly string _errorTextMessageKey = "TwoValuesAreNotEqualMessage";

    public CompareTextAttribute(string fieldToCompare, string currentField)
    {
        _fieldToCompare = fieldToCompare;
        _currentField = currentField;
    }

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (validationContext == null)
        {
            throw new ArgumentNullException(nameof(validationContext));
        }

        var currentValue = value as string;

        PropertyInfo? pInfo = validationContext.ObjectType.GetProperty(_fieldToCompare);
        if (pInfo == null)
        {
            return new ValidationResult($"Property '{_fieldToCompare}' does not exist.");
        }

        var compareValue = pInfo.GetValue(validationContext.ObjectInstance) as string;

        if (currentValue != compareValue)
        {
            return new ValidationResult(GetErrorMessage(validationContext));
        }

        return ValidationResult.Success;
    }

    private string GetErrorMessage(ValidationContext validationContext)
    {
        var localizer = validationContext.GetService(typeof(IStringLocalizer<SharedValidationAttribute>)) as IStringLocalizer<SharedValidationAttribute>;

        if (localizer != null && localizer[_errorTextMessageKey] != null)
        {
            return string.Format(localizer[_errorTextMessageKey], localizer[_currentField], localizer[_fieldToCompare]);
        }
        return $"Fields do not match.";
    }
}
```

## Email Requirements Attribute
``` cs title="EmailRequirements"
public class EmailRequirementsAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        var email = value as string;
        if (string.IsNullOrWhiteSpace(email))
        {
            return new ValidationResult(GetErrorMessage(validationContext, "EmailIsRequiredMessage"));
        }

        // Validate the email format using a simple regular expression
        if (!IsValid(email))
        {
            return new ValidationResult(GetErrorMessage(validationContext, "EmailIsInvalidMessage"));
        }

        return ValidationResult.Success;
    }

    private string GetErrorMessage(ValidationContext validationContext, string errorTextMessageKey)
    {
        var localizer = validationContext.GetService(typeof(IStringLocalizer<SharedValidationAttribute>)) as IStringLocalizer<SharedValidationAttribute>;

        if (localizer != null && localizer[errorTextMessageKey] != null)
        {
            return string.Format(localizer[errorTextMessageKey]);
        }

        return $"Email is required in a correct format.";
    }

    private bool IsValidEmail(string email)
    {
        // Regular expression pattern for validating an email address
        var pattern = @"^[^@\s]+@[^@\s]+\.[^@\s]+$";
        return Regex.IsMatch(email, pattern, RegexOptions.IgnoreCase);
    }

    public bool IsValid(string emailaddress)
    {
        try
        {
            MailAddress m = new MailAddress(emailaddress);

            return true;
        }
        catch (FormatException)
        {
            return false;
        }
    }
}
```

## Field Required Attribute
``` cs title="FieldIsRequired"
public class FieldIsRequiredAttribute : ValidationAttribute
{
    private readonly string _fieldToValidateName;
    private readonly string _errorTextMessageKey = "FieldIsRequiredMessage";

    public FieldIsRequiredAttribute(string fieldName)
    {
        _fieldToValidateName = fieldName;
    }

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (validationContext == null)
        {
            throw new ArgumentNullException(nameof(validationContext));
        }

        if (value == null || (value is string str && string.IsNullOrWhiteSpace(str)))
        {
            return new ValidationResult(GetErrorMessage(validationContext));
        }

        return ValidationResult.Success;
    }

    private string GetErrorMessage(ValidationContext validationContext)
    {
        var localizer = validationContext.GetService(typeof(IStringLocalizer<SharedValidationAttribute>)) as IStringLocalizer<SharedValidationAttribute>;

        if (localizer != null && localizer[_errorTextMessageKey] != null)
        {
            return string.Format(localizer[_errorTextMessageKey], localizer[_fieldToValidateName]);
        }
        return $"Field is required.";
    }
}
```

## Must Be True Attribute
``` cs title="MustBeTrue"
public class MustBeTrueAttribute : ValidationAttribute
{
    private readonly string _fieldToValidateName;
    private readonly string _errorTextMessageKey = "FieldMustBeTrueMessage";

    public MustBeTrueAttribute(string fieldName)
    {
        _fieldToValidateName = fieldName;
    }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (validationContext == null)
        {
            throw new ArgumentNullException(nameof(validationContext));
        }

        if (value == null || !(value is bool) || !(bool)value)
        {
            return new ValidationResult(GetErrorMessage(validationContext));
        }
        return ValidationResult.Success;
    }

    private string GetErrorMessage(ValidationContext validationContext)
    {
        var localizer = validationContext.GetService(typeof(IStringLocalizer<SharedValidationAttribute>)) as IStringLocalizer<SharedValidationAttribute>;

        if (localizer != null && localizer[_errorTextMessageKey] != null)
        {
            return string.Format(localizer[_errorTextMessageKey], localizer[_fieldToValidateName]);
        }
        return $"Field must be selected.";
    }
}
```

## Password Requirements Attribute
``` cs title="PasswordRequirements"
public class PasswordRequirementsAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        var localizer = validationContext.GetService(typeof(IStringLocalizer<PasswordRequirementsAttribute>)) as IStringLocalizer<PasswordRequirementsAttribute>;

        var password = value as string;
        if (string.IsNullOrWhiteSpace(password))
        {
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordIsRequiredMessage"));
        }

        bool hasDigit = password.Any(char.IsDigit);
        bool hasUpper = password.Any(char.IsUpper);
        bool hasLower = password.Any(char.IsLower);
        bool hasMinimumLength = password.Length >= 8;
        bool hasSpecialChar = password.Any(ch => !char.IsLetterOrDigit(ch));
        bool hasRequiredUniqueChars = password.Distinct().Count() >= 2;

        if (!hasDigit)
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordRequiresDigitMessage"));

        if (!hasUpper)
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordRequiresUpperCaseMessage"));

        if (!hasLower)
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordRequiresLowerCaseMessage"));

        if (!hasMinimumLength)
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordMinimumLengthMessage"));

        if (!hasSpecialChar)
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordRequiresSpecialCharMessage"));

        if (!hasRequiredUniqueChars)
            return new ValidationResult(GetErrorMessage(validationContext, "PasswordRequiresUniqueCharsMessage"));

        return ValidationResult.Success;
    }

    private string GetErrorMessage(ValidationContext validationContext, string errorTextMessageKey)
    {
        var localizer = validationContext.GetService(typeof(IStringLocalizer<SharedValidationAttribute>)) as IStringLocalizer<SharedValidationAttribute>;

        if (localizer != null && localizer[errorTextMessageKey] != null)
        {
            return string.Format(localizer[errorTextMessageKey]);
        }

        return $"Password is required.";
    }
}
```
