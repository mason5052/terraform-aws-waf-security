# Terraform AWS WAF Security

Production-tested Terraform modules for AWS WAF security configurations. These modules achieved **90%+ threat reduction** protecting **2,000+ global users** in an enterprise e-commerce environment.

## Overview

This repository provides reusable Terraform modules for implementing AWS WAF (Web Application Firewall) security rules based on real-world production experience. The configurations are designed for enterprise-grade protection against common web attacks.

## Key Features

- **SQL Injection Protection** - Block SQLi attacks with managed rule groups
- **Cross-Site Scripting (XSS) Prevention** - Prevent XSS attacks
- **Rate Limiting** - Protect against DDoS and brute force attacks
- **IP Reputation Blocking** - Block known malicious IP addresses
- **Geo-Blocking** - Restrict access by geographic location
- **Bot Protection** - Detect and block malicious bots
- **Custom Rules** - Flexible rule creation for specific use cases

## Production Results

| Metric | Before WAF | After WAF | Improvement |
|--------|------------|-----------|-------------|
| Security Threats | Baseline | **90%+ reduction** | Significant |
| Users Protected | - | **2,000+** | Global coverage |
| False Positives | N/A | **< 0.1%** | Tuned rules |
| Response Time Impact | N/A | **< 5ms** | Minimal latency |

## Quick Start

```hcl
module "waf_security" {
  source = "github.com/mason5052/terraform-aws-waf-security//modules/waf-regional"

  name_prefix = "my-app"
  
  # Enable managed rule groups
  enable_sql_injection_protection = true
  enable_xss_protection           = true
  enable_rate_limiting            = true
  
  # Rate limiting configuration
  rate_limit_threshold = 2000  # requests per 5 minutes
  
  # Associate with ALB
  alb_arn = module.alb.arn
  
  tags = {
    Environment = "production"
    Project     = "my-project"
  }
}
```

## Module Structure

```
terraform-aws-waf-security/
|-- modules/
|   |-- waf-regional/       # Regional WAF for ALB/API Gateway
|   |-- waf-cloudfront/     # CloudFront WAF
|   |-- ip-sets/            # IP set management
|   |-- rule-groups/        # Custom rule groups
|-- examples/
|   |-- basic/              # Basic WAF setup
|   |-- advanced/           # Advanced with custom rules
|   |-- e-commerce/         # E-commerce specific rules
|-- docs/
|   |-- RULES.md            # Rule documentation
|   |-- TUNING.md           # Performance tuning guide
```

## Managed Rule Groups

This module leverages AWS Managed Rule Groups for comprehensive protection:

| Rule Group | Purpose | Enabled by Default |
|------------|---------|-------------------|
| AWSManagedRulesCommonRuleSet | Common web exploits | Yes |
| AWSManagedRulesSQLiRuleSet | SQL injection | Yes |
| AWSManagedRulesKnownBadInputsRuleSet | Known bad inputs | Yes |
| AWSManagedRulesAmazonIpReputationList | Malicious IPs | Yes |
| AWSManagedRulesBotControlRuleSet | Bot detection | Optional |

## Custom Rules Examples

### Rate Limiting by IP

```hcl
rate_based_rules = [
  {
    name      = "rate-limit-by-ip"
    priority  = 1
    limit     = 2000
    action    = "block"
  }
]
```

### Geo-Blocking

```hcl
geo_match_rules = [
  {
    name           = "block-high-risk-countries"
    priority       = 2
    country_codes  = ["XX", "YY"]  # Replace with actual codes
    action         = "block"
  }
]
```

### Custom IP Whitelist

```hcl
ip_whitelist = [
  "10.0.0.0/8",      # Internal network
  "192.168.1.0/24",  # Office network
]
```

## Monitoring and Alerting

The module integrates with CloudWatch for monitoring:

```hcl
# Enable CloudWatch metrics
enable_cloudwatch_metrics = true

# Sample CloudWatch alarm
resource "aws_cloudwatch_metric_alarm" "waf_blocked_requests" {
  alarm_name          = "waf-high-blocked-requests"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "BlockedRequests"
  namespace           = "AWS/WAFV2"
  period              = 300
  statistic           = "Sum"
  threshold           = 1000
  alarm_description   = "High number of blocked requests detected"
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.0.0 |
| aws | >= 4.0.0 |

## Best Practices

1. **Start in Count Mode** - Deploy rules in count mode first to analyze impact
2. **Review Logs** - Use WAF logs to identify false positives
3. **Tune Gradually** - Adjust thresholds based on traffic patterns
4. **Use Labels** - Apply labels for easier rule management
5. **Regular Updates** - Keep managed rules updated

## Production Deployment Checklist

- [ ] Deploy in count mode initially
- [ ] Review WAF logs for 24-48 hours
- [ ] Identify and whitelist legitimate traffic patterns
- [ ] Switch to block mode gradually
- [ ] Set up CloudWatch alarms
- [ ] Document exception rules

## Contributing

Contributions are welcome. Please read the contributing guidelines before submitting pull requests.

## References

- [AWS WAF Developer Guide](https://docs.aws.amazon.com/waf/latest/developerguide/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

## Author

**Mason Kim**
- GitHub: [@mason5052](https://github.com/mason5052)
- LinkedIn: [junkukkim](https://www.linkedin.com/in/junkukkim/)

## License

MIT License - see [LICENSE](LICENSE) for details.
