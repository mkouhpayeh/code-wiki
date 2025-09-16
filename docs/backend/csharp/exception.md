# Custom Exception

```  cs Title="TestException.cs"
public class TestException : Exception
{
  public string Val;
  public TestException()
  {
    
  }
  public TestException(string message) : base(message)
  {
    
  }
  public TestException(string message, Exception innerException) : base(message, innerException)
  {
      
  }
  public TestException(string message, string val) : base(message)
  {
      Val = val;
  }
}
```

```  cs
try
{
  if (RegEx.IsMatch(value, @"\d")) throw new TestException("Value is a digit", value);
}
catch(TestException ex)
{
  return BadRequest(ex.Message, ex.Value);
}
```

